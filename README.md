# OBS Helper action 各功能使用示例
[OBS Helper action](https://github.com/marketplace/actions/huaweicloud-obs-helper)基于github workflow，实现：  
1、[上传文件/文件夹](#uploadSample)  
2、[下载文件/文件夹](#downloadSample)  
3、[创建桶](#createBucketSample)  
4、[删除桶](#deleteBucketSample)  

<p id="preparations">

# **前置工作**
1、需要开通华为云的OBS服务，并建好桶，[OBS主页](https://www.huaweicloud.com/product/obs.html)，[OBS文档](https://support.huaweicloud.com/obs/)；  
2、需要在项目的setting--Secret--Actions下添加华为云OBS服务的ACCESSKEY、SECRETACCESSKEY两个参数，[获取ak/sk方式](https://support.huaweicloud.com/api-obs/obs_04_0116.html)；  
3、注意替换参数region和参数bucket_name为自己OBS服务的真实region和桶名；  
# **参数说明**
### **对象操作参数**
**access_key**: 华为云账号的AK字符串，需要加密，请参照**前置工作**中的步骤2进行设置并使用，**必填**；  
**secret_key**：华为云账号的SK字符串，需要加密，请参照**前置工作**中的步骤2进行设置并使用，**必填**；  
**region**：OBS所在区域字符串，默认为*cn-north-4*，可以使用[目前OBS支持的region](#regionList)，**必填**；  
**bucket_name**：OBS的目标桶名，**必填**；  
**operation_type**：要进行的操作，上传请使用*upload*，下载请使用*download*，**必填**；  
**local_file_path**：对象的本地路径，上传对象时可填写多个，**必填**；  
**obs_file_path**：对象在桶内的路径，**必填**；  
**include_self_folder**：上传/下载文件夹时是否包含文件夹自身，默认为*false*，不填时默认不包含，**选填**；  
**exclude**：下载对象时，要排除的对象，上传时无用，不填时默认不排除，**选填**；  

### **桶操作参数**
**access_key**: 华为云账号的AK字符串，需要加密，请参照**前置工作**中的步骤2进行设置并使用，**必填**；  
**secret_key**：华为云账号的SK字符串，需要加密，请参照**前置工作**中的步骤2进行设置并使用，**必填**；  
**region**：OBS所在区域字符串，默认为*cn-north-4*，可以使用[目前OBS支持的region](#regionList)，**必填**；  
**bucket_name**：OBS的目标桶名，**必填**；  
**operation_type**：要进行的操作，创建桶请使用*createbucket*，删除桶请使用*deletebucket*，**必填**；  
**ACL**：创建桶时，桶的预定义访问策略，默认为*AclPrivate*（私有读写），可以使用[目前支持的预定义访问策略](#ACLList)，**选填**；  
**storage_class**： 创建桶时，桶的存储类型，默认为*StorageClassStandard*（标准存储），可以使用[目前支持的存储类型](#storageClassList)，**选填**；  
**clear_bucket**：删除桶时，是否清空桶内全部对象/碎片，默认为*true*，**选填**；  
### **参数支持列表**
<p id="regionList"></p>

目前OBS支持的区域名称和对应region如下
```text
  非洲-约翰内斯堡：  af-south-1	
  华北-北京四：	 cn-north-4	
  华北-北京一：	 cn-north-1	
  华北-乌兰察布一：  cn-north-9	
  华东-上海二：	 cn-east-2	
  华东-上海一：	 cn-east-3	
  华南-广州：	 cn-south-1	
  拉美-墨西哥城二：  la-north-2	
  拉美-墨西哥城一：  na-mexico-1	
  拉美-圣保罗一：  sa-brazil-1	
  拉美-圣地亚哥：  la-south-2	
  西南-贵阳一：	 cn-southwest-2	
  亚太-曼谷：	 ap-southeast-2	
  亚太-新加坡：	 ap-southeast-3	
  中国-香港：	 ap-southeast-1	
```
<p id="ACLList"></p>

目前OBS支持的预定义访问策略(ACL)如下
```text
  私有读写： AclPrivate
  公共读： AclPublicRead
  公共读写： AclPublicReadWrite
  桶公共读，桶内对象公共读： AclPublicReadDelivered
  桶公共读写，桶内对象公共读写： AclPublicReadWriteDelivered
```
<p id="storageClassList"></p>

目前OBS支持的存储类型(storage_class)如下
```text
  标准存储： StorageClassStandard
  低频访问存储： StorageClassWarm
  归档存储： StorageClassCold
```
# **对象操作使用样例**
<p id="uploadSample"></p>

## **上传对象使用样例**
假设您在cn-north-4的OBS桶内包含目录结构：
```text
  src
    └── upload
```
本地存在目录结构如下(即本仓库的resource/upload目录)：
```text
  resource
      └── upload
              ├── folder1
                      ├── file1-1.txt
                      └── file1-2.txt
              ├── folder2
                      ├── folder2-1
                              ├── folder2-1-1
                              └── file2-1-1.txt
                      └── file2-1.txt
              ├── file1.txt
              └── file2.txt
```
### **参数说明**
**access_key**: 华为云账号的AK字符串，需要加密，请参照[前置工作](#preparations)中的步骤2进行设置并使用，**必填**；  
**secret_key**：华为云账号的SK字符串，需要加密，请参照[前置工作](#preparations)中的步骤2进行设置并使用，**必填**；  
**region**：OBS所在区域字符串，默认为*cn-north-4*，可以使用[目前OBS支持的region](#regionList)，**必填**；  
**bucket_name**：OBS的目标桶名，**必填**；  
**operation_type**：要进行的操作，上传请使用*upload*；  
**local_file_path**：对象的本地路径，可填写多个，**必填**；  
**obs_file_path**：对象在桶内的路径，**必填**；  
**include_self_folder**：上传文件夹时是否包含文件夹自身，，不填时默认为*false*（不包含），**选填**；  
### 1、上传文件至OBS
上传单个文件时，obs_file_path参数以'/'结尾，代表将文件不重命名传入文件夹中；不以'/'结尾代表将文件以新名称上传至对应路径
#### 普通上传：
将本地文件resource/upload/file1.txt上传至桶内src/upload中
```yaml
        - name: Upload File To OBS
          uses: huaweicloud/obs-helper@v1.2.0
          id: upload_file_to_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: cn-north-4
            bucket_name: ${bucket_name}
            local_file_path: resource/upload/file1.txt
            obs_file_path: src/upload/
            operation_type: upload
```
上传成功后，您的OBS桶内目录结构应该为
```text
  src
    └── upload
            └── file1.txt
```
完整样例： [.github/workflows/upload-file-sample.yml](.github/workflows/upload-file-sample.yml)
#### 重命名上传：
将本地文件resource/upload/file1.txt上传至桶内src/upload并重命名文件为newFile1.txt
```yaml
    - name: Upload and Rename File To OBS
      uses: huaweicloud/obs-helper@v1.2.0
      id: upload_file_to_obs
      with:
        access_key: ${{ secrets.ACCESSKEY }}
        secret_key: ${{ secrets.SECRETACCESSKEY }}
        region: cn-north-4
        bucket_name: ${bucket_name}
        local_file_path: resource/upload/file1.txt
        obs_file_path: src/upload/newFile1.txt
        operation_type: upload
```
上传成功后，您的OBS桶内目录结构应该为
```text
  src
    └── upload
            └── newFile1.txt
```
完整样例： [.github/workflows/upload-file-rename-sample.yml](.github/workflows/upload-file-rename-sample.yml)
### 2、上传文件夹至OBS
#### 不包含根文件夹上传：
将本地文件夹resource/upload/folder2内的全部文件和文件夹上传至桶内src/upload/newFolder中
```yaml
        - name: Upload Folder To OBS
          uses: huaweicloud/obs-helper@v1.2.0
          id: upload_folder_to_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: cn-north-4
            bucket_name: ${bucket_name}
            local_file_path: resource/upload/folder2
            obs_file_path: src/upload/newFolder
            operation_type: upload
            include_self_folder: false     # 此时只上传了resource/upload/folder2内的文件和文件夹
```
因include_self_folder参数为false，所以此时待上传的文件和文件夹如下:
```text
  resource
      └── upload
              ├── folder1
                      ├── file1-1.txt
                      └── file1-2.txt
              ├── folder2
                      ├── folder2-1(待上传)
                              ├── folder2-1-1(待上传)
                              └── file2-1-1.txt(待上传)
                      └── file2-1.txt(待上传)
              ├── file1.txt
              └── file2.txt
```
上传成功后，您的OBS桶内目录结构应该为
```text
  src
    └── upload
            └── newFolder(自动创建)
                    ├── folder2-1
                            ├── folder2-1-1
                            └── file2-1-1.txt
                    └── file2-1.txt
```
完整样例： [.github/workflows/upload-folder-sample.yml](.github/workflows/upload-folder-sample.yml)
#### 包含根文件夹上传：
将本地文件夹resource/upload/folder2及其内的全部文件和文件夹上传至桶内src/upload/newFolder中
```yaml
        - name: Upload Folder To OBS
          uses: huaweicloud/obs-helper@v1.2.0
          id: upload_folder_to_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: cn-north-4
            bucket_name: ${bucket_name}
            local_file_path: resource/upload/folder2
            obs_file_path: src/upload/newFolder
            operation_type: upload
            include_self_folder: true    # 此时会将folder2文件夹也上传到src/upload/newFolder中
```
因include_self_folder参数为true，上传文件夹时包含文件夹自身，所以此时待上传的文件和文件夹如下:
```text
  resource
      └── upload
              ├── folder1
                      ├── file1-1.txt
                      └── file1-2.txt
              ├── folder2(待上传)
                      ├── folder2-1(待上传)
                              ├── folder2-1-1(待上传)
                              └── file2-1-1.txt(待上传)
                      └── file2-1.txt(待上传)
              ├── file1.txt
              └── file2.txt
```
上传成功后，您的OBS桶内目录结构应该为
```text
  src
    └── upload
            └── newFolder(自动创建)
                    └── folder2
                            ├── folder2-1
                                    ├── folder2-1-1
                                    └── file2-1-1.txt
                            └── file2-1.txt
```
完整样例： [.github/workflows/upload-folder-include-self-sample.yml](.github/workflows/upload-folder-include-self-sample.yml)
### 3、上传多个文件/文件夹至OBS
#### 将本地文件夹resource/upload/folder1、resource/upload/folder2，和本地文件resource/upload/file1.txt上传至桶内src/upload目录中
上传多文件/文件夹时，include_self_folder参数仅对文件夹有效，对文件无效，file1.txt在上传成功后的路径为src/upload/file1.txt
```yaml
        - name: Upload Folder and File To OBS
          uses: huaweicloud/obs-helper@v1.2.0
          id: upload_mult_files_to_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: cn-north-4
            bucket_name: ${bucket_name}
            local_file_path: |
              resource/upload/folder1
              resource/upload/folder2
              resource/upload/file1.txt
            obs_file_path: src/upload 
            operation_type: upload
            include_self_folder: true
```
因include_self_folder参数为true，所以此时待上传的文件和文件夹如下：
```text
  resource
      └── upload
              ├── folder1(待上传)
                      ├── file1-1.txt(待上传)
                      └── file1-2.txt(待上传)
              ├── folder2(待上传)
                      ├── folder2-1(待上传)
                              ├── folder2-1-1(待上传)
                              └── file2-1-1.txt(待上传)
                      └── file2-1.txt(待上传)
              ├── file1.txt(待上传)
              └── file2.txt
```
上传成功后，您的OBS桶内目录结构应该为
```text
  src
    └── upload
            ├── folder1
                    ├── file1-1.txt
                    └── file1-2.txt
            ├── folder2
                    ├── folder2-1
                            ├── folder2-1-1
                            └── file2-1-1.txt
                    └── file2-1.txt
            └── file1.txt
```
完整样例： [.github/workflows/upload-mult-files-sample.yml](.github/workflows/upload-mult-files-sample.yml)

<p id="downloadSample"></p>

## **下载对象使用样例**
**注意**：下载时需要保证下载到一个存在的本地目录，即local_file_path中的每一级文件夹都是存在的。  

假设您的OBS桶内包含目录结构：
```text
  src
    └── download
            ├── obsFolder1
                    ├── obsFile1-1.txt
                    └── obsFile1-2.txt
            ├── obsFolder2
                    └── obsFile2-1.txt
            └── obsFile1.txt
```
并且本地存在目录(即本仓库的resource/download目录)：
```text
  resource
      └── download
              └── obsFile2-1.txt(文件夹)
                     └──  localFile.txt
```
### **参数说明**
**access_key**: 华为云账号的AK字符串，需要加密，请参照[前置工作](#preparations)中的步骤2进行设置并使用，**必填**；  
**secret_key**：华为云账号的SK字符串，需要加密，请参照[前置工作](#preparations)中的步骤2进行设置并使用，**必填**；  
**region**：OBS所在区域字符串，默认为*cn-north-4*，可以使用[目前OBS支持的region](#regionList)，**必填**；  
**bucket_name**：OBS的目标桶名，**必填**；  
**operation_type**：要进行的操作，下载请使用*download*，**必填**；  
**local_file_path**：对象的本地路径，上传对象时可填写多个，**必填**；  
**obs_file_path**：对象在桶内的路径，**必填**；  
**include_self_folder**：下载文件夹时是否包含文件夹自身，默认为*false*（不包含），**选填**；  
**exclude**：下载时，要排除的对象，不填时默认不排除任何对象，**选填**；  
### 1、从OBS下载文件
下载文件时，首先会检查本地是否存在local_file_path代表的文件/文件夹，  
如果不存在此文件/文件夹，会尝试将文件下载为文件local_file_path；  
如果存在此文件，则会覆盖此文件下载；  
如果存在此文件夹，则会尝试将文件下载至此文件夹中，若此文件夹中仍有和目标文件同名的文件夹，则本次下载会失败。  
具体示例如下：
#### 普通下载：
下载obs中的文件src/download/obsFile1.txt至本地resource/download目录下
```yaml
        - name: Download File From OBS
          uses: huaweicloud/obs-helper@v1.2.0
          id: download_file_from_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: cn-north-4
            bucket_name: ${bucket_name}
            obs_file_path: src/download/obsFile1.txt
            local_file_path: resource/download/
            operation_type: download
```
下载成功后，本地的目录结构应该为：
```text
    resource
        └── download
                ├── obsFile2-1.txt
                        └──  localFile.txt
                └── obsFile1.txt(此次下载的文件)
```
1).本地存在文件夹'resource'、'resource/download';  
2).'resource/download'文件夹中不存在名为'obsFile1.txt'的文件夹;  
所以最终obs上的对象'src/download/obsFile1.txt'会下载为本地文件'resource/download/obsFile1.txt'  

完整样例： [.github/workflows/download-file-sample.yml](.github/workflows/download-file-sample.yml)
#### 重命名下载：
下载obs中的文件src/download/obsFile1.txt至本地resource/download目录，并重命名为file3.txt
```yaml
        - name: Download and Rename File From OBS
          uses: huaweicloud/obs-helper@v1.2.0
          id: download_file_from_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: cn-north-4
            bucket_name: ${bucket_name}
            obs_file_path: src/download/obsFile1.txt
            local_file_path: resource/download/file3.txt
            operation_type: download
```
下载成功后，本地的目录结构应该为：
```text
    resource
        └── download
                ├── obsFile2-1.txt
                        └── localFile.txt
                └── file3.txt(此次下载的文件)
```
1).本地文件夹'resource'、'resource/download'都存在;  
2).'resource/download'文件夹中不存在名为'file3.txt'的文件夹;  
所以最终obs上的对象'src/download/obsFile1.txt'会下载为本地文件'resource/download/file3.txt'  

完整样例： [.github/workflows/download-file-rename-sample.yml](.github/workflows/download-file-rename-sample.yml)
#### 特殊情景-本地目录存在与待下载文件同名的文件夹：
下载obs中的文件src/download/obsFolder2/obsFile2-1.txt至本地resource/download目录
```yaml
        - name: Download File From OBS
          uses: huaweicloud/obs-helper@v1.2.0
          id: download_file_from_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: cn-north-4
            bucket_name: ${bucket_name}
            obs_file_path: src/download/obsFolder2/obsFile2-1.txt
            local_file_path: resource/download
            operation_type: download
```
下载成功后，本地的目录结构应该为
```text
    resource
        └── download
                └── obsFile2-1.txt
                        ├── localFile.txt
                        └── obsFile2-1.txt(此次下载的文件)
```
1).本地文件夹'resource'、'resource/download'都存在;  
2).'resource/download'中存在和待下载文件同名的文件夹'obsFile2-1.txt';  
3).文件夹'resource/download/obsFile2-1.txt'中不存在文件夹obsFile2-1.txt;  
所以最终文件会下载为resource/download/obsFile2-1.txt/obsFile2-1.txt；  
Tips：如果文件夹'resource/download/obsFile2-1.txt'中仍然存在文件夹'obsFile2-1.txt'，则此次下载会失败。  

完整样例： [.github/workflows/download-file-special-sample.yml](.github/workflows/download-file-special-sample.yml)
### 2、从OBS下载文件夹
下载文件夹时，参数*exclude*中的对象在obs不存在时，不会影响本次下载。  
#### 普通下载：
下载obs中的src/download文件夹下的内容到本地目录resource/download中
```yaml
        - name: Download Folder From OBS
          uses: huaweicloud/obs-helper@v1.2.0
          id: download_folder_from_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: cn-north-4
            bucket_name: ${bucket_name}
            obs_file_path: src/download
            local_file_path: resource/download
            operation_type: download
```
此时待下载的文件和文件夹如下:
```text
  src
    └── download
            ├── obsFolder1(待下载)
                    ├── obsFile1-1.txt(待下载)
                    └── obsFile1-2.txt(待下载)
            ├── obsFolder2(待下载)
                    └── obsFile2-1.txt(待下载)
            └── obsFile1.txt(待下载)
```
下载成功后，本地的目录结构应该为
```text
  resource
      └── download
              ├── obsFolder1(此次下载的文件夹)
                      ├── obsFile1-1.txt（此次下载的文件）
                      └── obsFile1-2.txt（此次下载的文件）
              ├── obsFolder2(此次下载的文件夹)
                      └── obsFile2-1.txt（此次下载的文件）
              ├── obsFile1.txt（此次下载的文件）
              └── obsFile2-1.txt
                      └── localFile.txt
```
完整样例： [.github/workflows/download-folder-sample.yml](.github/workflows/download-folder-sample.yml)
#### 排除下载：
下载obs中的src/download文件夹及其内容到本地目录resource/download中，并排除src/upload/folder1文件夹和src/upload/folder2/file2-1.txt
```yaml
        - name: Download Folder Exclude Some Objects From OBS
          uses: huaweicloud/obs-helper@v1.2.0
          id: download_folder_from_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: cn-north-4
            bucket_name: ${bucket_name}
            obs_file_path: src/download
            local_file_path: resource/download
            operation_type: download
            include_self_folder: true
            # 下载时排除的文件/文件夹，可一次排除多个路径
            exclude: |
              src/download/obsFolder1/obsFile1-1.txt
              src/download/obsFolder2
```
因参数include_self_folder为true，并且排除了文件'src/download/obsFolder1/obsFile1-1.txt'和文件夹'src/download/obsFolder2'，所以此时待下载的文件和文件夹如下:
```text
  src
    └── download(待下载)
            ├── obsFolder1(待下载)
                    ├── obsFile1-1.txt
                    └── obsFile1-2.txt(待下载)
            ├── obsFolder2
                    └── obsFile2-1.txt
            └── obsFile1.txt(待下载)
```
下载成功后，本地的目录结构应该为
```text
  resource
      └── download
              ├── download(此次下载的文件夹)
                      ├── obsFolder1(此次下载的文件夹)
                              └── obsFile1-2.txt（此次下载的文件）
                      └── obsFile1.txt（此次下载的文件）
              └── obsFile2-1.txt
                      └── localFile.txt
```
完整样例： [.github/workflows/download-folder-exculde-objects-sample.yml](.github/workflows/download-folder-exculde-objects-sample.yml)

# **桶操作使用样例**
OBS桶命名规则：  
3～63个字符，数字或字母开头，支持**小写字母**、**数字**、“**-**”、“**.**”  
禁止使用类IP地址（如255.255.255.0）  
禁止以“-”或“.”开头及结尾  
禁止两个“.”相邻（如：“my..bucket”）  
禁止“.”和“-”相邻（如：“my-.bucket”和“my.-bucket”） 

<p id="createBucketSample"></p>

## **创建桶使用样例**
### **参数说明**
**access_key**: 华为云账号的AK字符串，需要加密，请参照[前置工作](#preparations)中的步骤2进行设置并使用，**必填**；  
**secret_key**：华为云账号的SK字符串，需要加密，请参照[前置工作](#preparations)中的步骤2进行设置并使用，**必填**；  
**region**：OBS所在区域字符串，默认为*cn-north-4*，可以使用[目前OBS支持的region](#regionList)，**必填**；  
**bucket_name**：OBS的目标桶名，**必填**；  
**operation_type**：要进行的操作，创建桶请使用*createbucket*，**必填**；  
**ACL**：创建桶时，桶的预定义访问策略，默认为*AclPrivate*（私有读写），可以使用[目前支持的预定义访问策略](#ACLList)，**选填**；  
**storage_class**： 创建桶时，桶的存储类型，默认为*StorageClassStandard*（标准存储），可以使用[目前支持的存储类型](#storageClassList)，**选填**；  
### 默认创建： 
假设您的OBS中不存在名为'bucket-test'的桶   
```yaml
        - name: Create Default Bucket on OBS
          uses: huaweicloud/obs-helper@v1.2.0
          id: create_default_bucket_on_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: cn-north-4
            operationType: createbucket
            bucket_name: bucket-test
```
执行成功后，您的OBS中会新增一个名为'bucket-test'的桶，桶的预定义访问策略为**私有读写**，存储类型**标准存储**

完整样例： [.github/workflows/create-bucket-default.yml](.github/workflows/create-bucket-default.yml)
### 指定预定义访问策略/存储类型：
假设您的OBS中不存在名为'bucket-test'的桶   
```yaml
        - name: Create Bucket on OBS
          uses: huaweicloud/obs-helper@v1.2.0
          id: create_bucket_on_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: cn-north-4
            operationType: createbucket
            bucket_name: bucket-test
            ACL: AclPublicReadWrite
            storage_class: StorageClassWarm
```
执行成功后，您的OBS中会新增一个名为'bucket-test'的桶，桶的预定义访问策略为**公共读写**，存储类型**低频访问存储**

完整样例： [.github/workflows/create-bucket.yml](.github/workflows/create-bucket.yml)

<p id="deleteBucketSample"></p>

## **删除桶使用样例**
桶为空时，桶的拥有者可以根据需要删除桶，以免占用桶数量配额。  
桶为空包含两方面含义：  
1).桶内没有任何对象或对象的任何历史版本。  
2).桶内没有任何未合并的多段上传任务，即桶内不存在碎片。  
  
删除桶时，action**默认**会执行如下步骤：  
1).判断桶是否为空，若桶非空，清空桶
2).删除桶  
若不允许action执行清空步骤，请设置参数*clear_bucket*为false。此时如果桶不为空，则会删除失败并提示桶不为空。
### **参数说明**
**access_key**: 华为云账号的AK字符串，需要加密，请参照[前置工作](#preparations)中的步骤2进行设置并使用，**必填**；  
**secret_key**：华为云账号的SK字符串，需要加密，请参照[前置工作](#preparations)中的步骤2进行设置并使用，**必填**；  
**region**：OBS所在区域字符串，默认为*cn-north-4*，可以使用[目前OBS支持的region](#regionList)，**必填**；  
**bucket_name**：OBS的目标桶名，**必填**；  
**operation_type**：要进行的操作，删除桶请使用*deletebucket*，**必填**；  
**clear_bucket**：删除桶时，是否清空桶内全部对象/碎片，默认为*true*（清空），**选填**；  
### 清空桶内对象+删除桶：
假设您的OBS中存在名为'bucket-test'的桶
```yaml
        - name: Delete Bucket on OBS
          uses: huaweicloud/obs-helper@v1.2.0
          id: delete_bucket_on_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: cn-north-4
            operationType: deletebucket
            bucket_name: bucket-test
```
执行成功后，桶'bucket-test'会被清空并删除。

完整样例： [.github/workflows/delete-bucket-default.yml](.github/workflows/delete-bucket-default.yml)  
### 不清空桶内对象+删除桶
假设您的OBS中存在名为'bucket-test'的桶
```yaml
        - name: Delete Bucket on OBS
          uses: huaweicloud/obs-helper@v1.2.0
          id: delete_bucket_on_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: cn-north-4
            operationType: deletebucket
            bucket_name: bucket-test
            clear_bucket: false
```
执行后，如果桶'bucket-test'为空，桶会被删除；若不为空，则会执行失败，桶和桶内对象均不会被删除。

完整样例： [.github/workflows/delete-bucket-no-clear.yml](.github/workflows/delete-bucket-no-clear.yml)  

点击跳转至Action：[HuaweiCloud Obs Helper](https://github.com/marketplace/actions/huaweicloud-obs-helper)