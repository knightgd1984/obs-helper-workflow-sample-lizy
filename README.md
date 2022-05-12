# 上传对象至OBS/从OBS下载对象示例
基于github workflow，实现：
1、上传文件/文件夹至OBS
2、从OBS下载文件/文件夹

## **前置工作**
1、需要开通华为云的OBS服务，并建好桶，[OBS主页](https://www.huaweicloud.com/product/obs.html)，[OBS文档](https://support.huaweicloud.com/obs/)；
2、需要在项目的setting--Secret--Actions下添加华为云OBS服务的ACCESSKEY、SECRETACCESSKEY两个参数，[获取ak/sk方式](https://support.huaweicloud.com/api-obs/obs_04_0116.html)；
3、注意替换参数region和参数bucket_name为自己OBS服务的真实region和桶名；

## **参数说明**
**access_key**: 华为云账号的AK字符串，需要加密，请参照**前置工作**中的步骤2进行设置并使用，**必填**；
**secret_key**：华为云账号的SK字符串，需要加密，请参照**前置工作**中的步骤2进行设置并使用，**必填**；
**region**：OBS所在区域字符串，如'cn-north-4'，**必填**；
**bucket_name**：OBS的目标桶名，**必填**；
**operation_type**：要进行的操作，上传请使用'upload'，下载请使用'download'，**必填**；
**local_file_path**：对象的本地路径，上传对象时可填写多个，**必填**；
**obs_file_path**：对象在桶内的路径，**必填**；
**include_self_folder**：上传/下载文件夹时是否包含文件夹自身，可使用true、false及其对应大写字符，不填时默认不包含，**选填**；
**exclude**：下载时要排除的对象，上传时无用，不填时默认不排除，**选填**；

## **上传使用样例**
假设您的OBS桶内包含目录结构：
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
### 1、上传文件至OBS
上传单个文件时，obs_file_path参数以'/'结尾，代表将文件不重命名传入文件夹中；不以'/'结尾代表将文件以新名称上传至对应路径
#### 将本地文件resource/upload/file1.txt上传至桶内src/upload中
```yaml
        - name: Upload File To OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: upload_file_to_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: ${region}
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
完整样例： .github/workflows/upload-file-sample.yml
#### 将本地文件resource/upload/file1.txt上传至桶内src/upload并重命名文件为newFile1.txt
```yaml
    - name: Upload and Rename File To OBS
      uses: huaweicloud/obs-helper@v1.1.0
      id: upload_file_to_obs
      with:
        access_key: ${{ secrets.ACCESSKEY }}
        secret_key: ${{ secrets.SECRETACCESSKEY }}
        region: ${region}
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
完整样例： .github/workflows/upload-file-rename-sample.yml
### 2、上传文件夹至OBS
#### 将本地文件夹resource/upload/folder2内的全部文件和文件夹上传至桶内src/upload/newFolder中
```yaml
        - name: Upload Folder To OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: upload_folder_to_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: ${region}
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
完整样例： .github/workflows/upload-folder-sample.yml
#### 将本地文件夹resource/upload/folder2及其内的全部文件和文件夹上传至桶内src/upload/newFolder中
```yaml
        - name: Upload Folder To OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: upload_folder_to_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: ${region}
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
完整样例： .github/workflows/upload-folder-include-self-sample.yml
### 3、上传多个文件至OBS
#### 将本地文件夹resource/upload/folder1、resource/upload/folder2，和本地文件resource/upload/file1.txt上传至桶内src/upload目录中
上传多文件/文件夹时，include_self_folder参数仅对文件夹有效，对文件无效，file1.txt在上传成功后的路径为src/upload/file1.txt
```yaml
        - name: Upload Folder and File To OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: upload_mult_files_to_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: ${region}
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
完整样例： .github/workflows/upload-mult-files-sample.yml
## **下载使用样例**
注意：下载时需要保证下载到一个存在的本地目录，即local_file_path中的每一级文件夹都是存在的。

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
### 1、从OBS下载文件
下载文件时，首先会检查本地是否存在local_file_path代表的文件/文件夹，
如果不存在此文件/文件夹，会尝试将文件下载为文件local_file_path；
如果存在此文件，则会覆盖此文件下载；
如果存在此文件夹，则会尝试将文件下载至此文件夹中，若此文件夹中仍有和目标文件同名的文件夹，则本次下载会失败。
具体示例如下：
#### 下载obs中的文件src/download/obsFile1.txt至本地resource/download目录下
```yaml
        - name: Download and From OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: download_file_from_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: ${region}
            bucket_name: ${bucket_name}
            obs_file_path: src/download/obsFile1.txt
            local_file_path: resource/download
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

完整样例： .github/workflows/download-file-sample.yml
#### 下载obs中的文件src/download/obsFile1.txt至本地resource/download目录，并重命名为file3.txt
```yaml
        - name: Download and Rename File From OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: download_file_from_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: ${region}
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

完整样例： .github/workflows/download-file-rename-sample.yml
#### 下载obs中的文件src/download/obsFolder2/obsFile2-1.txt至本地resource/download目录
```yaml
        - name: Download and Rename File From OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: download_file_from_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: ${region}
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

完整样例： .github/workflows/download-file-special-sample.yml
### 2、从OBS下载文件夹
下载文件夹时，若obs不存在exclude参数中的对象时，不会影响本次下载。
#### 下载obs中的src/download文件夹下的内容到本地目录resource/download中
```yaml
        - name: Download Folder From OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: download_folder_from_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: ${region}
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
完整样例： .github/workflows/download-folder-sample.yml
#### 下载obs中的src/download文件夹及其内容到本地目录resource/download中，并排除下载src/upload/folder1文件夹和src/upload/folder2/file2-1.txt
```yaml
        - name: Download Folder Exclude Some Objects From OBS
          uses: huaweicloud/obs-helper@v1.1.0
          id: download_folder_from_obs
          with:
            access_key: ${{ secrets.ACCESSKEY }}
            secret_key: ${{ secrets.SECRETACCESSKEY }}
            region: ${region}
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
完整样例： .github/workflows/download-folder-exculde-objects-sample.yml