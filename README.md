# 上传对象至OBS/从OBS下载对象示例
基于github workflow，结合官方的action完成如下工作：
1、上传文件/文件夹至OBS
2、从OBS下载文件/文件夹

## **前置工作**
1、需要开通华为云的OBS服务，并建好桶，OBS主页:https://www.huaweicloud.com/product/obs.html，OBS文档:https://support.huaweicloud.com/obs/
2、需要在项目的setting--Secret--Actions下添加华为云OBS服务的ACCESSKEY,SECRETACCESSKEY两个参数,获取ak/sk方式:https://support.huaweicloud.com/api-obs/obs_04_0116.html
3、注意替换region和bucket_name为自己OBS服务的真实region和桶名

## **参数说明**
access_key: 华为云账号的AK字符串，必填；
secret_key：华为云账号的SK字符串，必填；
region：OBS的终端节点字符串，如'cn-north-4'，必填；
bucket_name：OBS的目标桶名，必填；
operation_type：要进行的操作，上传文件请使用'upload'，下载文件请使用'download'，必填；
local_file_path：对象的本地路径，上传对象时可填写多个，必填；
obs_file_path：对象在桶内的路径，必填；
include_self_folder：上传/下载文件夹时是否包含文件夹自身，可使用yes/y/true、no/n/false及其对应大写方式，不填时默认不包含，选填；
exclude：下载时要排除的对象，上传时无用，选填，不填时默认不排除；

## **上传使用样例**
假设您的OBS桶内包含目录结构：
```text
  src
    └── upload
```
### 1、上传文件至OBS
上传单个文件时，obs_file_path参数以'/'结尾，代表将文件不重命名传入文件夹中；不以'/'结尾代表将文件以新名称上传至对应路径
完整样例： .github/workflows/upload-file-sample.yml
#### 将本地文件resource/file1.txt上传至桶内src/upload并重命名文件为newFile1.txt
```yaml
        - name: Upload and Rename File To OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: upload_file_to_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: region
            bucket_name: bucket_name
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
#### 将本地文件resource/file1.txt上传至桶内src/upload中
```yaml
        - name: Upload File To OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: upload_file_to_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: region
            bucket_name: bucket_name
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
### 2、上传文件夹至OBS
完整样例： .github/workflows/upload-folder-sample.yml
#### 将本地文件夹resource/folder2内的全部文件和文件夹上传至桶内src/upload/newFolder中
```yaml
        - name: Upload Folder To OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: upload_folder_to_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: region
            bucket_name: bucket_name
            local_file_path: resource/upload/folder2
            obs_file_path: src/upload/newFolder
            operation_type: upload
            include_self_folder: no     # 此时只上传了resource/folder2内的文件和文件夹
```
上传成功后，您的OBS桶内目录结构应该为
```text
  src
    └── upload
            └── newFolder
                    ├── folder2-1
                            ├── folder2-1-1
                            └── file2-1-1.txt
                    └── file2-1.txt
```
#### 将本地文件夹resource/folder2及其内的全部文件和文件夹上传至桶内src/upload/newFolder中
```yaml
        - name: Upload Folder To OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: upload_folder_to_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: region
            bucket_name: bucket_name
            local_file_path: resource/upload/folder2
            obs_file_path: src/upload/newFolder
            operation_type: upload
            include_self_folder: yes    # 此时会将folder2文件夹也上传到src/upload/newFolder中
```
上传成功后，您的OBS桶内目录结构应该为
```text
  src
    └── upload
            └── newFolder
                    └── folder2
                            ├── folder2-1
                                    ├── folder2-1-1
                                    └── file2-1-1.txt
                            └── file2-1.txt
```
### 3、上传多个路径至OBS
完整样例： .github/workflows/upload-mult-files-sample.yml
#### 将本地文件夹resource/folder1、resource/folder2，和本地文件resource/file1.txt上传至桶内src/upload目录中
上传多文件/文件夹时，include_self_folder参数仅对文件夹有效，对文件无效，file1.txt在上传成功后的路径为src/upload/file1.txt
```yaml
        - name: Upload Folder and File To OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: upload_mult_files_to_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: region
            bucket_name: bucket_name
            local_file_path: |
                              resource/upload/folder1
                              resource/upload/folder2
                              resource/upload/file1.txt
            obs_file_path: src/upload 
            operation_type: upload
            include_self_folder: yes
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
## **下载使用样例**
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
### 1、从OBS下载文件
下载单个文件时，若输入的local_file_path是一个存在的文件夹，则会将文件直接下载至此文件夹中，若此文件夹中有和文件同名的文件夹，则会取消本次下载。
若local_file_path不是一个存在的文件夹，则会将待下载的文件重命名并下载。
完整样例： .github/workflows/download-file-sample.yml
#### 下载obs中的src/upload/obsFile1.txt至本地resource/download目录，并重命名为file3.txt
```yaml
        - name: Download and Rename File From OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: download_file_from_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: region
            bucket_name: bucket_name
            obs_file_path: src/download/obsFile1.txt
            # 因本地resource/download目录下不存在名为file3.txt的文件夹，所以最终文件会下载为resource/download/file3.txt
            local_file_path: resource/download/file3.txt
            operation_type: download
```
下载成功后，本地生成的目录结构应该为
```text
    └── resource
            └── download
                    └── file3.txt
```
#### 一个较为特殊的例子
```yaml
        - name: Download and Rename File From OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: download_file_from_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: region
            bucket_name: bucket_name
            obs_file_path: src/download/obsFolder2/obsFile2-1.txt
            local_file_path: resource/download
            operation_type: download
```
下载成功后，本地生成的目录结构应该为
```text
    └── resource
            └── download
                    └── obsFile2-1.txt
                            └── obsFile2-1.txt
```
拆开来看：
1).因为本地存在文件夹resource/download，所以等同于未重命名下载文件到resource/download文件夹中，即local_file_path: resource/download/obsFile2-1.txt；
2).此时因本地路径resource/download中存在和文件同名的文件夹obsFile2-1.txt，并且文件夹resource/download/obsFile2-1.txt中不存在文件夹obsFile2-1.txt，所以最终文件会下载为resource/download/obsFile2-1.txt/obsFile2-1.txt；
判断2的逻辑只会判断一次，即如果文件夹resource/download/obsFile2-1.txt中仍然存在文件夹obsFile2-1.txt，则此次下载会失败。
### 2、从OBS下载文件夹
下载文件夹时，注意要下载到一个本地存在的路径，即local_file_path是一个存在的文件夹。
exclude参数中的对象在obs不存在时，不会影响本次下载。
完整样例： .github/workflows/download-folder-sample.yml
#### 下载obs中的src/download文件夹及其内容到本地目录resource/download中，并排除下载src/upload/folder1文件夹和src/upload/folder2/file2-1.txt
```yaml
        - name: Download Folder From OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: download_folder_from_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: region
            bucket_name: bucket_name
            obs_file_path: src/download
            local_file_path: resource/download
            operation_type: download
            include_self_folder: yes    # 若为yes/y/true，会下载upload文件夹及其内容；否则只下载upload文件夹内的内容
            # 下载时排除的文件/文件夹，可一次排除多个路径
            exclude: |
                      src/download/obsFolder1/obsFile1-1.txt
                      src/download/obsFolder2
```
下载成功后，本地生成的目录结构应该为
```text
    └── resource
            └── download
                    └── download
                            ├── obsFolder1
                                    └── obsFile1-2.txt
                            └── obsFile1.txt
```
#### 下载obs中的src/download文件夹下的内容到本地目录resource/download中，并排除下载src/upload/abc.rar（此时obs上不存在的对象）
```yaml
        - name: Download Folder From OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: download_folder_from_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: region
            bucket_name: bucket_name
            obs_file_path: src/download
            local_file_path: resource/download
            operation_type: download
            exclude: src/upload/abc.rar
```
下载成功后，本地生成的目录结构应该为
```text
    └── resource
            └── download
                    ├── obsFolder1
                            └── obsFile1-1.txt
                            └── obsFile1-2.txt
                    ├── obsFolder2
                            └── obsFile2-1.txt
                    └── obsFile1.txt
```