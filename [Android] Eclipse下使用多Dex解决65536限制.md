### Dex 65536
>UNEXPECTED TOP-LEVEL EXCEPTION: java.lang.IllegalArgumentException: method ID not in [0, 0xffff]: 65536

这个报错说明应用中的Dex 文件方法数超过了最大值65536的上限
### 为什么会引起这种错误
在Android系统中，App的所有的代码编译后的二进制文件都在Dex文件里面；而早期设计的时候，会把每个类的方法id存在一个链表结构里，这个链表的长度是用一个short类型保存的，导致方法数量不能超过65536个。尽管在新版本的Android系统中，修复了这个问题，但是我们仍然需要对低版本的Android系统做兼容。
### 如何解决
#### 常用的解决方法
1. 删除掉无用的jar包和代码，或是在 project.proterty 中加入`dex.force.jumbo=true`,这样就可以成功编译了，但是在2.3及更低的版本系统上安装还是会出现`INSTALL_FAILED_DEXOPT`异常。
2. 应用插件化，可以使用一些开源的插件化框架，比如淘宝的[Dexposed](https://github.com/AiAndroid/dexposed)，支付宝的[AndFix](https://github.com/alibaba/AndFix), 及类似Qzone的[Nawa](https://github.com/jasonross/Nuwa) 等
3. 分割 Dex，将一些不经常更新的 jar 合并后转成一个 dex，通过 DexClassLoader 加载进来；

#### Eclipse 中多 Dex 的解决方案
概述：将不经常更新的 jar，编译到一个dex当中并作为资源放在 assets 文件夹中，应用启动的时候通过 DexClassLoader 动态载入。
##### 合并 jar 包的 build 文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project name="b" basedir="/Users/dongxu/Desktop/build/build5" default="mergeJar">

<target name="mergeJar"  description="description">  
                                <jar destfile="merge.jar">
                                        <zipfileset src="rxandroid-1.2.0.jar"/>
                                        <zipfileset src="rxjava-1.0.10.jar"/>
                                </jar>  
                            </target>  

</project>
```
`ant -buildfile build.xml` 执行 build，并生成 merge.jar。

#####将 jar 文件转成 dex 二进制文件
cmd 进入到 android-sdks/build-tools/x.x.x 文件夹，然后执行`dx --dex --output=merge.dex merge.jar`，获取 merge.dex 并保存到 assets 中。

##### 修改 build.xml
如果是 ant 脚本打包，需要自定义 build.xml，并做如下修改保证编译通过

```
    <property name="project.root.dir" location="" />
    <property name="android.platforms.dir" value="${sdk.dir}/platforms/${target}" />
	<property name="jar.android" value="${android.platforms.dir}/android.jar" />
    <property name="jar.data" value="${project.root.dir}/libs/dx/data.jar" />
 
    
        <!-- Compiles this project's .java files into .class files. -->
    <target name="-compile" depends="-pre-build, -build-setup, -code-gen, -pre-compile">
        <do-only-if-manifest-hasCode elseText="hasCode = false. Skipping...">
            <!-- merge the project's own classpath and the tested project's classpath -->
            <path id="project.javac.classpath">
                <path refid="project.all.jars.path" />
                <path refid="tested.project.classpath" />
                <path path="${java.compiler.classpath}" />
            </path>
            <javac encoding="${java.encoding}"
                    source="${java.source}" target="${java.target}"
                    debug="true" extdirs="" includeantruntime="false"
                    destdir="${out.classes.absolute.dir}"
                    bootclasspath="${jar.android};${jar.data}"
                    verbose="${verbose}"
                    classpathref="project.javac.classpath"
                    fork="${need.javac.fork}">
                <src path="${source.absolute.dir}" />
                <src path="${gen.absolute.dir}" />
                <compilerarg line="${java.compilerargs}" />
            </javac>

            <!-- if the project is instrumented, intrument the classes -->
            <if condition="${build.is.instrumented}">
                <then>
                    <echo level="info">Instrumenting classes from ${out.absolute.dir}/classes...</echo>

                    <!-- build the filter to remove R, Manifest, BuildConfig -->
                    <getemmafilter
                            appPackage="${project.app.package}"
                            libraryPackagesRefId="project.library.packages"
                            filterOut="emma.default.filter"/>

                    <!-- define where the .em file is going. This may have been
                         setup already if this is a library -->
                    <property name="emma.coverage.absolute.file" location="${out.absolute.dir}/coverage.em" />

                    <!-- It only instruments class files, not any external libs -->
                    <emma enabled="true">
                        <instr verbosity="${verbosity}"
                               mode="overwrite"
                               instrpath="${out.absolute.dir}/classes"
                               outdir="${out.absolute.dir}/classes"
                               metadatafile="${emma.coverage.absolute.file}">
                            <filter excludes="${emma.default.filter}" />
                            <filter value="${emma.filter}" />
                        </instr>
                    </emma>
                </then>
            </if>

            <!-- if the project is a library then we generate a jar file -->
            <if condition="${project.is.library}">
                <then>
                    <echo level="info">Creating library output jar file...</echo>
                    <property name="out.library.jar.file" location="${out.absolute.dir}/classes.jar" />
                    <if>
                        <condition>
                            <length string="${android.package.excludes}" trim="true" when="greater" length="0" />
                        </condition>
                        <then>
                            <echo level="info">Custom jar packaging exclusion: ${android.package.excludes}</echo>
                        </then>
                    </if>

                    <propertybyreplace name="project.app.package.path" input="${project.app.package}" replace="." with="/" />

                    <jar destfile="${out.library.jar.file}">
                        <fileset dir="${out.classes.absolute.dir}"
                                includes="**/*.class"
                                excludes="${project.app.package.path}/R.class ${project.app.package.path}/R$*.class ${project.app.package.path}/BuildConfig.class"/>
                        <fileset dir="${source.absolute.dir}" excludes="**/*.java ${android.package.excludes}" />
                    </jar>
                </then>
            </if>

        </do-only-if-manifest-hasCode>
    </target>
```
`bootclasspath="${jar.android};${jar.data}"` 编译过程中需要依赖的类，并且不会导入到 dex 当中。

##### DexClassLoader 加载 dex
自定义 application，载入 assets 中的 dex 文件

```
public class App extends Application {
	@Override
    public void onCreate() {
        super.onCreate();
        dexTool();
    }
    
	private void loadDex() {

		String dexName = "merge.dex";
        File dexDir = new File(getFilesDir(), "dex");
        dexDir.mkdir();
        File dexFile = new File(dexDir, dexName);
        File dexOpt = new File(dexDir, "opt");
        dexOpt.mkdir();
        try {
            InputStream ins = getAssets().open(dexName);
            if (dexFile.length() != ins.available()) {
                FileOutputStream fos = new FileOutputStream(dexFile);
                byte[] buf = new byte[4096];
                int l;
                while ((l = ins.read(buf)) != -1) {
                    fos.write(buf, 0, l);
                }
                fos.close();
            }
            ins.close();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

        ClassLoader cl = getClassLoader();
        DexClassLoader dcl = new DexClassLoader(dexFile.getAbsolutePath(),
                dexOpt.getAbsolutePath(), null, cl.getParent());

        try {
            Field f = ClassLoader.class.getDeclaredField("parent");
            f.setAccessible(true);
            f.set(cl, dcl);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
｝
```
参考：

[eclipse下android项目解决方法数id超过65535](http://my.oschina.net/u/992018/blog/354513)

[Dex65536](https://github.com/mmin18/Dex65536)

