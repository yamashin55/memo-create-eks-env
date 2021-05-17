# memo-create-eks-env 　
※ EKSの検証環境を構築するメモです。

## 事前インストール

(Windowsで環境作ると大変。後述するWorkstation環境が楽)  

Windows10の場合は以下をインストール
  - [vscode](https://code.visualstudio.com/)
  - [ms-vscode-remote.vscode-remote-extensionpack](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)
  - [docker](https://www.docker.com/)
  - [wsl2](https://docs.microsoft.com/en-us/windows/wsl/) on windows workstations

インストールしたらVSCode起動して、
  - コマンドパレットでRemote-Containersに接続  
    ```
    Remote-Containers: Rebuild and Reopen in Container
    ```

 - VSCode上でターミナル起動して、awsctl、terraformをインストール

    [awscli install](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/install-cliv2-linux.html)
    ```
    $ aws --version

    aws-cli/1.19.53 Python/3.8.5 Linux/5.4.72-microsoft-standard-WSL2 botocore/1.20.53
    ```

    [terraform install](https://learn.hashicorp.com/tutorials/terraform/install-cli)
    ```
    $ terraform -version

    Terraform v0.14.10
    ```

    [kubectl install](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/)
    ```
    $ kubectl version --client

    Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.7", 
    ```
---
## EKS 環境作る

1. gitでCloneする
      ```
      git clone -b 'v1.1.0' --single-branch https://github.com/f5devcentral/f5-digital-customer-engagement-center
      ```
1. ディレクトリ移動
    ```
    cd ~/f5-digital-customer-engagement-center/solutions/delivery/application_delivery_controller/nginx/kic/aws
    ```
1. admin.auto.tfvarsのコピー
    ```
    cp admin.auto.tfvars.example admin.auto.tfvars
    ```
1. admin.auto.tfvarsの編集
    ```
    adminAccountName = "zadmin"  
    resourceOwner    = "syamada"  
    awsRegion        = "us-east-1"  
    awsAz1           = "us-east-1a"  
    awsAz2           = "us-east-1f"  
    sshPublicKey     = "ssh-rsa AAAAB3NzaC1.........."  
    ```
1. 環境構築
    ```
    ./setup.sh
    ```
1. 
1. 
1. 
1. 
1. 
1. 
1. 
1. 
1. 


