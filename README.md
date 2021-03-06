# BTAutoPacking-iOS
Auto packing for iOS.  
iOS自动化打包程序，只需配置项目路径，超简单!
支持多target打包

# Usage
```
$sh BTAutoPacking-iOS
```
# Source Code
```
#!/bin/sh

# auto packing for iOS
# shell+xcodebuild packing for iOS
# by biostome 2019-5-19

# 说明
# 此脚本目前只针对有工作空间workspace文件的打包。
# 此脚本需要您的项目可以手动打包成功一次，只有xcode手动打包成功过一次，自动打包会省去很多配置内容。
# 此脚本力求最简化配置，最好是只需要配置一处，但目前还是需要配置两处：
    #1：匹配项目目录
    #2：设置Plist文件
# 请全局搜索【配置】配置内容。

# 【配置】项目目录
__project_path="/Users/XXXXX/Desktop/AutoPacking-iOS/AutoPackingDemo"

#—————————————— 目录地址 --------------
__auto_packing_path=`pwd`

# 【检查和配置】请修改Plist/AdHocExportOptionsPlist.plist文件，这个文件可以通过手动打包方式导出一次就会自动生成，复制出来用就可以了，当然也可以自己手动新建。
# 这里用的是adhoc打包方式，需要其他的打包方式请自己生成plist文件，然后修改本指令Plist文件名。
__export_options_plist="${__auto_packing_path}/Plist/AdHocExportOptionsPlist.plist"

# ipa包到处地址，导出位置是脚本下的ipas目录，没有ipas目录会自动生成
__exportPath="${__auto_packing_path}/ipas"

# 归档archive地址，没有则自动生成
__build_path="${__auto_packing_path}/Build/"


#——————————————旧文件删除————————————————
echo "<-----------------🚀旧文件删除🚀------------------->"
if [ -d "${__build_path}" ] ; then
    echo "发现Build文件夹有文件，即将清空文件夹📁"
    rm -rf ${__build_path}/*
fi

if [ -d "${__exportPath}" ] ; then
    echo "发现ipas文件夹有文件，即将清空文件夹📁"
    rm -rf ${__exportPath}/*
fi


#-------------- 打包前准备 -----------------
echo "<-----------------🚀打包前准备🚀------------------->"
# cd 到 projectPath 这个路径
cd ${__project_path}

# 获取Project名称
__project_name=`ls $__project_path | grep .xcodeproj | sed 's/.xcodeproj//g'`
#__project_name=`find . -name *.xcodeproj | awk -F "[/.]" '{print $(NF-1)}'`

# 获取workspace名称
__work_space_name="${__project_name}"

# 【配置】请注意配置Targets，可以手动配置也可以通过自动获取（搜索【自动设置Targets】）
# 自动设置会自动找到项目中所有的Target，全部一起打包。
# 不要一起打包的话，请手动设置。

# 手动设置Targets
#__targets=("A scheme" "B scheme" "C scheme")

# 自动设置Targets

# 方式一
__targets=`xcodebuild -list -project ${__project_name}.xcodeproj -json | awk '/\"schemes/,/],/' | sed s/[[:space:]]//g | sed -e '1d' -e '$d' -e 's/,//g' -e 's/\"//g'`

# 方式二
#__targets=`xcodebuild -list -project ${__project_name}.xcodeproj -json | grep - | sed s/,//g | sed s/\"//g | sed s/[[:space:]]//g`

# 方式四
#__targets=`xcodebuild -list -workspace ${__work_space_name}.xcworkspace | grep [-] | grep -v Pods | sed s/[[:space:]]//g | awk '{{printf"%s ",$0}}'`


#-------------- 检查数据,检查这里待完善 -----------------
echo "<-----------------🚀检查数据🚀------------------->"
echo "方案：${__targets}"


#-------------- 开始打包过程 -----------------
echo "<-----------------🚀开始打包🚀------------------->"

#clean所有target项目
xcodebuild clean -configuration Release -alltargets
for scheme in ${__targets[@]}; do
    # 编译 ".xcarchive" 文件的存放地址
    __archivePath="${__build_path}/${scheme}"
    echo "<-----------------🚀开始归档archive文件🚀------------------->"
    # 归档 对应手动打包archive（使用workspace）
    xcodebuild archive -workspace ${__work_space_name}.xcworkspace -scheme ${scheme} -configuration Release -archivePath ${__archivePath}.xcarchive
    # 检查是否构建成功
    # xcarchive 实际是一个文件夹不是一个文件所以使用 -d 判断
    if [ -d "${__archivePath}.xcarchive" ] ; then
        echo "项目构建成功 🎉 🎉 🎉"
    else
        echo "项目构建失败 😢 😢 😢"
        exit 1
    fi
    # 生成打包时间
    __time=$(date "+_%Y-%m-%d_%H-%M-%S")
    __ipaName="${scheme}${__time}"
    # 拼接导出ipa的地址
    __ipaPath="${__exportPath}/${__ipaName}"
    echo "<-----------------🚀开始导出ipa文件🚀------------------->"
    # 对应导出步骤
    xcodebuild -exportArchive -archivePath ${__archivePath}.xcarchive -exportPath ${__ipaPath} -exportOptionsPlist ${__export_options_plist}

    if [ -d "${__ipaPath}" ] ; then
        echo "导出 ${__ipaName}.ipa 包成功 🎉 🎉 🎉"
    else
        echo "导出 ${__ipaName}.ipa 包失败 😢 😢 😢"
    fi
done

echo "<-----------------结束------------------->"
echo "打包总耗时: ${SECONDS}s"
```
# Contributing
Fork it!
Create your feature branch: `git checkout -b my-new-feature`  
Commit your changes: `git commit -am 'Add some feature'`  
Push to the branch: `git push origin my-new-feature`  
Submit a pull request :D  

# Contact me
E-mail: 453816118@qq.com  
Blog: https://www.jianshu.com/u/285d083f8c60

# License
SwiftExtensions is released under the MIT license. See LICENSE for details.
