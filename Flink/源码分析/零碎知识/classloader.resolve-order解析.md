在使用 Apache Flink 进行流处理时，类加载器（ClassLoader）的配置是一个重要但常常被忽视的环节。Flink 提供了一个配置选项 `classloader.resolve-order`，用于控制类加载器的类加载顺序

#### 什么是 `classloader.resolve-order`

`classloader.resolve-order` 是 Flink 配置文件 `flink-conf.yaml` 中的一个选项，用于控制 Flink 在加载类时的优先级。这个选项决定了 Flink 是优先从用户代码的类加载器加载类，还是优先从系统类加载器加载类。

`classloader.resolve-order` 有两个可选值：

1. **child-first**（默认）
2. **parent-first**

#### child-first 模式

当 `classloader.resolve-order` 设置为 `child-first` 时，Flink 会优先从用户代码的类加载器中加载类。如果用户代码的类加载器中没有找到该类，Flink 才会从系统类加载器中加载。

**虽然允许优先从用户代码加载器中加载，但也设置了保护机制，对于一些flink自身的类及其他，默认从parent-classloader加载**

- **优点**：
    - 允许用户自定义类优先于系统类加载。
    - 有助于避免与 Flink 自身**依赖的库**发生冲突。
- **适用场景**：
    - 当用户的应用程序使用了与 Flink 相同的库，但需要不同版本时。

#### parent-first 模式

当 `classloader.resolve-order` 设置为 `parent-first` 时，Flink 会优先从系统类加载器中加载类。如果系统类加载器中没有找到该类，Flink 才会从用户代码的类加载器中加载。

- **优点**：
    - 确保系统类优先加载，避免可能的类冲突。
    - 更符合 Java 的类加载机制，通常更稳定。
- **适用场景**：
    - 当用户应用程序依赖于与 Flink 兼容的库版本时。

### 如何配置 `classloader.resolve-order`

在 Flink 的 `flink-conf.yaml` 配置文件中，可以设置 `classloader.resolve-order`：

```
classloader.resolve-order: child-first
```

或

```
classloader.resolve-order: parent-first
```

### 源码分析

```JAVA
package org.apache.flink.runtime.execution.librarycache;

import org.apache.flink.configuration.CoreOptions;
import org.apache.flink.util.ChildFirstClassLoader;
import org.apache.flink.util.FlinkUserCodeClassLoader;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.Closeable;
import java.io.IOException;
import java.net.URL;
import java.net.URLClassLoader;
import java.util.Enumeration;
import java.util.function.Consumer;

/** Gives the URLClassLoader a nicer name for debugging purposes. */
public class FlinkUserCodeClassLoaders {

    private FlinkUserCodeClassLoaders() {}

    public static URLClassLoader parentFirst(
            URL[] urls,
            ClassLoader parent,
            Consumer<Throwable> classLoadingExceptionHandler,
            boolean checkClassLoaderLeak) {
        FlinkUserCodeClassLoader classLoader =
                new ParentFirstClassLoader(urls, parent, classLoadingExceptionHandler);
        return wrapWithSafetyNet(classLoader, checkClassLoaderLeak);
    }
		//alwaysParentFirstPatterns 默认的保护策略，
  	/**
  	    public static final ConfigOption<String> ALWAYS_PARENT_FIRST_LOADER_PATTERNS =
            ConfigOptions.key("classloader.parent-first-patterns.default")
                    .defaultValue(
   "java.;scala.;org.apache.flink.;com.esotericsoftware.kryo;org.apache.hadoop.;javax.annotation.;org.xml;javax.xml;org.apache.xerces;org.w3c;org.rocksdb.;"
                                    + PARENT_FIRST_LOGGING_PATTERNS)
                   
  	**/
    public static URLClassLoader childFirst(
            URL[] urls,
            ClassLoader parent,
            String[] alwaysParentFirstPatterns,
            Consumer<Throwable> classLoadingExceptionHandler,
            boolean checkClassLoaderLeak) {
        FlinkUserCodeClassLoader classLoader =
                new ChildFirstClassLoader(
                        urls, parent, alwaysParentFirstPatterns, classLoadingExceptionHandler);
        return wrapWithSafetyNet(classLoader, checkClassLoaderLeak);
    }

  	//根据配置获取对应的classloader加载策略
    public static URLClassLoader create(
            ResolveOrder resolveOrder,
            URL[] urls,
            ClassLoader parent,
            String[] alwaysParentFirstPatterns,
            Consumer<Throwable> classLoadingExceptionHandler,
            boolean checkClassLoaderLeak) {

        switch (resolveOrder) {
            case CHILD_FIRST:
                return childFirst(
                        urls,
                        parent,
                        alwaysParentFirstPatterns,
                        classLoadingExceptionHandler,
                        checkClassLoaderLeak);
            case PARENT_FIRST:
                return parentFirst(
                        urls, parent, classLoadingExceptionHandler, checkClassLoaderLeak);
            default:
                throw new IllegalArgumentException(
                        "Unknown class resolution order: " + resolveOrder);
        }
    }

    private static URLClassLoader wrapWithSafetyNet(
            FlinkUserCodeClassLoader classLoader, boolean check) {
        return check
                ? new SafetyNetWrapperClassLoader(classLoader, classLoader.getParent())
                : classLoader;
    }

    /** Class resolution order for Flink URL {@link ClassLoader}. */
    public enum ResolveOrder {
        CHILD_FIRST,
        PARENT_FIRST;

        public static ResolveOrder fromString(String resolveOrder) {
            if (resolveOrder.equalsIgnoreCase("parent-first")) {
                return PARENT_FIRST;
            } else if (resolveOrder.equalsIgnoreCase("child-first")) {
                return CHILD_FIRST;
            } else {
                throw new IllegalArgumentException("Unknown resolve order: " + resolveOrder);
            }
        }
    }

    /**
     * Regular URLClassLoader that first loads from the parent and only after that from the URLs.
     */
    public static class ParentFirstClassLoader extends FlinkUserCodeClassLoader {

        ParentFirstClassLoader(
                URL[] urls, ClassLoader parent, Consumer<Throwable> classLoadingExceptionHandler) {
            super(urls, parent, classLoadingExceptionHandler);
        }

        static {
            ClassLoader.registerAsParallelCapable();
        }
    }

    /**
     * Ensures that holding a reference on the context class loader outliving the scope of user code
     * does not prevent the user classloader to be garbage collected (FLINK-16245).
     *
     * <p>This classloader delegates to the actual user classloader. Upon {@link #close()}, the
     * delegate is nulled and can be garbage collected. Additional class resolution will be resolved
     * solely through the bootstrap classloader and most likely result in ClassNotFound exceptions.
     */
    private static class SafetyNetWrapperClassLoader extends URLClassLoader implements Closeable {
        private static final Logger LOG =
                LoggerFactory.getLogger(SafetyNetWrapperClassLoader.class);

        private volatile FlinkUserCodeClassLoader inner;

        SafetyNetWrapperClassLoader(FlinkUserCodeClassLoader inner, ClassLoader parent) {
            super(new URL[0], parent);
            this.inner = inner;
        }

        @Override
        public void close() {
            final FlinkUserCodeClassLoader inner = this.inner;
            if (inner != null) {
                try {
                    inner.close();
                } catch (IOException e) {
                    LOG.warn("Could not close user classloader", e);
                }
            }
            this.inner = null;
        }

        private FlinkUserCodeClassLoader ensureInner() {
            if (inner == null) {
                throw new IllegalStateException(
                        "Trying to access closed classloader. Please check if you store "
                                + "classloaders directly or indirectly in static fields. If the stacktrace suggests that the leak "
                                + "occurs in a third party library and cannot be fixed immediately, you can disable this check "
                                + "with the configuration '"
                                + CoreOptions.CHECK_LEAKED_CLASSLOADER.key()
                                + "'.");
            }
            return inner;
        }

        @Override
        public Class<?> loadClass(String name) throws ClassNotFoundException {
            return ensureInner().loadClass(name);
        }

        @Override
        protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
            // called for dynamic class loading
            return ensureInner().loadClass(name, resolve);
        }

        @Override
        public URL getResource(String name) {
            return ensureInner().getResource(name);
        }

        @Override
        public Enumeration<URL> getResources(String name) throws IOException {
            return ensureInner().getResources(name);
        }

        @Override
        public URL[] getURLs() {
            return ensureInner().getURLs();
        }

        static {
            ClassLoader.registerAsParallelCapable();
        }
    }
}

```



