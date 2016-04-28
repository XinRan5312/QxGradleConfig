# QxGradleConfig
借助简书上的一篇文章  总结了下  Android的gradle配置  需要待续
http://www.jianshu.com/p/9b50c4059d61
一、构建变体
1. BuildType
1.1默认buildType
buildTypes {
    release {
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
}
// release版本中设置了开启混淆，并且定义了混淆文件的位置
默认情况下还有一个debug版本，我们也可以添加对debug版本的一些设置

buildTypes {
    debug {
        minifyEnabled false
    }
}
// debug版本中关闭混淆
1.2自定义buildType
除了默认的构建版本，还可以创建自己的构建版本

buildTypes {
    custom.initWith(buildTypes.debug)
    custom {
        applicationIdSuffix  ".custom"
        versionNameSuffix  "-custom"    }}
// custom使用initWith方法复制debug版本并创建了一个新的构建版本，相当于继承了debug版本
// custom版本中添加applicationId后缀，添加versionName后缀
其他属性的设置可以查看buildType的文档。

2. ProductFlavor
productFlavor用来为app创建不同的版本，如：免费版和付费版、不同应用市场的渠道包等。
创建方式：

android {
    productFlavors {
        free { // 免费版
        }
        paid { // 付费版
        }
        wandoujia { // 豌豆荚应用市场渠道包
        }
        myapp { // 应用宝应用市场渠道包
        }
    }
}
每一个生产版本都可以设置applicationId、versionCode、versionName等许多属性，具体可以查看productFlavors的文档。

3. BuildVariant
buildType和productFlavor相结合，组成了构建变体。每创建一个buildType或productFlavor，都会同时创建相应的变体。例如：创建一个myapp的productFlavor时，将会创建出两个相应的变体：myappDebug和myappRelease。

二、依赖管理
依赖管理的具体细节这里不多说明，想要详细了解的可以查看依赖管理的文档。

1. 添加依赖
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:23.2.1'
}
2. 依赖配置
Gradle中的依赖会以配置（configurations）分组，一个配置就是一组依赖，称之为依赖配置
android插件定义了一些标准的配置，如：

compile：编译项目代码所需要的依赖。
debugCompile：debug版本编译项目代码所需要的依赖，对应buildTypes中的debug。默认情况下，包含了编译时产生的类文件，以及编译时期所需要的依赖。
releaseCompile：release版本编译项目代码所需要的依赖，对应buildType中的release。默认情况下，包含了编译时产生的类文件，以及编译时期所需要的依赖。
testCompile：编译测试代码时所需要的依赖。默认情况下，包含了编译时产生的类文件，以及编译时期所需要的依赖。
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:23.3.0'
    testCompile 'junit:junit:4.12'
}
三、多版本不同依赖配置
1. 需求
接入广告是目前许多APP的盈利方式，但是随着不同平台的的广告SDK越来越多，各大应用市场也开始加强这方面的监管了，有的应用市场禁止接入广告SDK，有的应用市场则要求只能接入特定的广告SDK，所以就需要针对不同市场生成含有不同广告SDK的应用版本。

2. 实现
2.1 原理
每创建一个BuildType就会自动创建一个基于它名字的编译依赖配置<buildType>Compile
每创建一个ProductFlavor就会自动创建一个基于它名字的编译依赖配置<productFlavor>Compile
所以利用这个特点，可以为不同的版本设置不同的依赖配置。
2.2 依赖配置语法说明
依赖jar文件
dependencies{
    // 依赖某个jar文件
    complie files('libs/xxx.jar') 
    // 依赖libs目录下所有以.jar结尾的文件
    complie fileTree(dir: 'libs', include: ['*.jar']) 
    // 依赖libs目录下除了xxx.jar以外的所有以.jar结尾的文件
    complie fileTree(dir: 'libs', exclude: ['xxx.jar'], include: ['*.jar']) 
}
依赖module
dependencies{
    // 依赖本地项目工程下的某个module
    complie project(:'moduleName')
}
依赖aar文件
android {
    repositories {
        flatDir {
            dirs 'libs'
        }
    }
}
dependencies{
    // 依赖名字为xxx后缀为aar的文件
    compile (name: 'xxx', ext: 'aar')
}
依赖远程库
dependencies{
    // 格式为：group:name:version
    compile 'com.android.support:appcompat-v7:23.3.0'
    // 排除某个传递依赖
    compile 'com.android.support:appcompat-v7:23.3.0'{
        exclude group: 'xxx', module: 'xxx'
}
2.3 依赖jar包实现不同依赖
默认不依赖任何广告sdk
在build.gradle中添加ProductFlavor，如下：

android {
    productFlavors {
        baidu { // 百度应用市场
        }
        lenovo { // 联想应用市场
        }
        common { // 其他不监管广告市场
        }
    }
}
将所有广告sdk的jar包统一添加前缀"ad_"，在build.gradle中的dependencies下添加

dependencies{
    // 默认不添加广告sdk
    compile fileTree(dir: 'libs', excludes: ['ad_*.jar'], include: ['*.jar'])
    // 百度市场添加百度广告sdk
    baiduCompile files(libs/ad_baidu_sdk.jar)
    // 联想应用市场的包添加联想广告sdk
    lenovoCompile files(libs/ad_lenovo_sdk.jar)
    // 其他不监管广告应用市场添加通用的广告sdk
    commonCompile files(libs/ad_common_sdk.jar)
}
配置完成后运行gradle assembleRelease命令，就会生成各个市场的渠道包，并且含有不用的广告sdk。

2.4 依赖远程库实现不同依赖
方法一：默认不依赖任何广告sdk
在build.gradle中添加ProductFlavor，如下：
android {
    productFlavors {
        baidu { // 百度应用市场
        }
        lenovo { // 联想应用市场
        }
        common { // 其他不监管广告市场
        }
    }
}
在build.gradle中的dependencies下添加

dependencies{
    // 百度市场添加百度广告sdk
    baiduCompile "com.baidu:ad:1.0.0"
    // 联想应用市场的包添加联想广告sdk
    lenovoCompile "com.lenovo:ad:1.0.0"
    // 其他不监管广告应用市场添加通用的广告sdk
    commonCompile "com.common:ad:1.0.0"
}
方法二：默认依赖通用广告sdk
在build.gradle中添加ProductFlavor，如下：
android {
    productFlavors {
        baidu { // 百度应用市场
        }
        lenovo { // 联想应用市场
        }
        huawei { // 华为应用市场
        }
    }
}
在build.gradle中的dependencies下添加

dependencies{
    // 默认添加通用广告sdk
    compile "com.common:ad:1.0.0"
    // 百度市场在默认的compile中去掉通用广告sdk
    baiduCompile configurations.compile {
        exclude(group: "com.common:ad:1.0.0", module: "ad")
    }
    // 百度市场添加百度广告sdk
    baiduCompile "com.baidu:ad:1.0.0"
    // 联想市场在默认的compile中去掉通用广告sdk
    baiduCompile configurations.compile {
        exclude(group: "com.common:ad:1.0.0", module: "ad")
    }
    // 联想应用市场的包添加联想广告sdk
    lenovoCompile "com.lenovo:ad:1.0.0"
    // 华为市场在默认的compile中去掉通用广告sdk
    baiduCompile configurations.compile {
        exclude(group: "com.common:ad:1.0.0", module: "ad")
    }
}

文／zly394（简书作者）
原文链接：http://www.jianshu.com/p/9b50c4059d61
著作权归作者所有，转载请联系作者获得授权，并标注“简书作者”。
