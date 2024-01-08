# #实现zip压缩

### zip压缩

```java
public class ZipUtils {

    public static void main(String[] args) throws IOException {
        // generatorZip("D:/project-info/control_center/src/main/java", "C:\\Users\\hxe\\Desktop\\zip\\yundun.zip");
        generatorZip("D:/software-info/eclipse", "C:\\Users\\hxe\\Desktop\\zip\\eclipse.zip");
    }

    /**
     * 将指定目录生成压缩文件
     * <pre>
     *  2、线程池设置最大线程数量公式：
     *
     * （1）IO密集型： (√)
     *
     * 线程数 = 核数 * 期望CPU利用率 * 总时间 (CPU计算时间 + 等待时间) / CPU 计算时间
     *
     * 例如：4核CPU计算时间是50%，其它等待时间是50%，期望cpu被100%利用，套用公式
     * 4 * 100% * 100% / 50% = 8
     * 例如：4核CPU计算时间是10%，其它等待时间是90%，期望cpu被100%利用，套用公式
     * 4 * 100%* 100% / 10%= 40
     *
     * （2）CPU密集型应用，则线程池大小设置为 N+1
     *
     * IO密集型经验应用，线程池大小设置为 2N+1 (N为CPU数量，下同)
     * </pre>
     *
     * @param sourcePath 待压缩目录
     * @param targetPath 压缩文件路径,形如@{"C:\\Users\\xxxx\\Desktop\\zip\\xxxxx.zip"}
     */
    public static void generatorZip(String sourcePath, String targetPath) {
        long st = System.currentTimeMillis();
        List<String> paths = getAbsolutePath(sourcePath, "");
        // paths.stream().forEach(System.out::println);
        try (ZipOutputStream zip = new ZipOutputStream(new FileOutputStream(new File(targetPath))); BufferedOutputStream bos = new BufferedOutputStream(zip)) {
            for (String suffix : paths) {
                zip.putNextEntry(new ZipEntry(suffix));
                try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(new File(sourcePath + File.separator + suffix)))) {
                    int len;
                    byte[] bytes = new byte[1024];
                    while ((len = bis.read(bytes)) != -1) {
                        bos.write(bytes, 0, len);
                    }
                }
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            System.out.println("压缩耗费时间：" + (System.currentTimeMillis() - st) + "ms");
        }
    }

    private static List<String> getAbsolutePath(String rootPath, String childPath) {
        File root = new File(childPath == null || "".equals(childPath) ? rootPath : childPath);
        List<String> paths = new ArrayList<>();
        if (root.isDirectory()) {
            for (File child : root.listFiles()) {
                if (child.isDirectory()) {
                    paths.addAll(getAbsolutePath(rootPath, child.getPath()));
                } else {
                    paths.add(getRelativePath(rootPath, child.getPath()));
                }
            }
        } else {
            paths.add(getRelativePath(rootPath, root.getPath()));
        }
        return paths;
    }

    private static String getRelativePath(String prefix, String path) {
        if (prefix == null || "".equals(prefix)) {
            return path;
        }
        return path.substring(path.indexOf(prefix) + prefix.length() + 2);
    }
}
```

优化：利用`apache-commons`​并行压缩

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-compress</artifactId>
    <version>1.20</version>
</dependency>
```

实现如下

```java
public class ZipUtils {
	public static void main(String[] args) throws IOException {
        // generatorZip("D:/project-info/control_center/src/main/java", "C:\\Users\\hxe\\Desktop\\zip\\yundun.zip");
        generatorZip("D:/software-info/eclipse", "C:\\Users\\hxe\\Desktop\\zip\\eclipse.zip");
    }

    /**
     * 将指定目录生成压缩文件
     * <pre>
     *  2、线程池设置最大线程数量公式：
     *
     * （1）IO密集型： (√)
     *
     * （2）CPU密集型应用，则线程池大小设置为 N+1
     *
     * IO密集型经验应用，线程池大小设置为 2N+1 (N为CPU数量，下同)
     * </pre>
     *
     * @param sourcePath 待压缩目录
     * @param targetPath 压缩文件路径,形如@{"C:\\Users\\xxxx\\Desktop\\zip\\xxxxx.zip"}
     */
    public static void generatorZip(String sourcePath, String targetPath) {
        long st = System.currentTimeMillis();
        List<String> paths = getAbsolutePath(sourcePath, "");
        System.out.println("read directory cost(" + (System.currentTimeMillis() - st) + ")ms");
        // paths.stream().forEach(System.out::println);
        ExecutorService executorService = Executors.newFixedThreadPool((int) (Runtime.getRuntime().availableProcessors() / 0.5));
        ParallelScatterZipCreator parallelScatterZipCreator = new ParallelScatterZipCreator(executorService);
        try (ZipArchiveOutputStream zipArchiveOutputStream = new ZipArchiveOutputStream(new FileOutputStream(new File(targetPath)))) {
            zipArchiveOutputStream.setEncoding("UTF-8");
            for (String suffix : paths) {
                File inFile = new File(sourcePath + File.separator + suffix);
                ZipArchiveEntry zipArchiveEntry = new ZipArchiveEntry(suffix);
                zipArchiveEntry.setMethod(ZipArchiveEntry.DEFLATED);
                zipArchiveEntry.setSize(inFile.length()); // 未压缩数据大小
                zipArchiveEntry.setUnixMode(UnixStat.FILE_FLAG | 436);
                parallelScatterZipCreator.addArchiveEntry(zipArchiveEntry, () -> {
                    try {
                        return new FileInputStream(inFile);
                    } catch (FileNotFoundException e) {
                        return new NullInputStream(0);
                    }
                });
            }
            parallelScatterZipCreator.writeTo(zipArchiveOutputStream);
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            System.out.println("压缩耗费时间：" + (System.currentTimeMillis() - st) + "ms");
        }
    }

    private static List<String> getAbsolutePath(String rootPath, String childPath) {
        File root = new File(childPath == null || "".equals(childPath) ? rootPath : childPath);
        List<String> paths = new ArrayList<>();
        if (root.isDirectory()) {
            for (File child : root.listFiles()) {
                if (child.isDirectory()) {
                    paths.addAll(getAbsolutePath(rootPath, child.getPath()));
                } else {
                    paths.add(getRelativePath(rootPath, child.getPath()));
                }
            }
        } else {
            paths.add(getRelativePath(rootPath, root.getPath()));
        }
        return paths;
    }

    private static String getRelativePath(String prefix, String path) {
        if (prefix == null || "".equals(prefix)) {
            return path;
        }
        return path.substring(path.indexOf(prefix) + prefix.length() + 2);
    }
}
```
