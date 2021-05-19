# memo-create-eks-env 　
※ EKSの検証環境を構築するメモです。

## 事前インストール
※Windowsで作業環境作ると大変  
※とりあえずTerraform, AWS CTLがあれば下記は飛ばして良い  

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
    ※作成まで、およそ20分程  
    ※エラーが出たら、   ```terraform init --upgrade```  

    ![/setup.sh](./images/1.png)  

    主な作成オブジェクト
    - VPC  
    - Subnet  
    - Route Table  
    - Internet Gateway  
    - Security Group  
    - EC2  
    - EIP  
    - EKS Cluster  
    - ECR Private Registriy

1. ECRをコンテナリポジトリとして利用するために作成されたECRを登録する。

    「us-west-2」と「ecrRepositoryURL」は自分のに合わせて変更する。  

    aws ecr get-login-password --region ***us-west-2*** | docker login --username AWS --password-stdin ***ecrRepositoryURL***  

    ```
    $ aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 673420076654.dkr.ecr.us-east-1.amazonaws.com/demo-cluster-ecr-2372788194
    ```

1. Kube-configをアップデート

    「us-west-2」と「kubernetesClusterName」は自分のに合わせて変更する。  

    aws eks --region ***us-west-2*** update-kubeconfig --name ***kubernetesClusterName***

    ```
    aws eks --region us-east-1 update-kubeconfig --name demo-cluster-2372788194
    ```

1. ELBを利用できるようにパブリックサブネットに2つのタグを配置する

    「publicSubnetAZ1」　「publicSubnetAZ2」　「kubernetesClusterName」は自分のに合わせて変更する。  

    aws ec2 create-tags \
    --resources ***publicSubnetAZ1*** ***publicSubnetAZ2*** \  
    --tags Key=kubernetes.io/cluster/***kubernetesClusterName***,Value=shared Key=kubernetes.io/role/elb,Value=1


    ```
    aws ec2 create-tags \
    --resources subnet-0706ed6f210c9c338 subnet-017d2335b672300bd \
    --tags Key=kubernetes.io/cluster/demo-cluster-2372788194,Value=shared   Key=kubernetes.io/role/elb,Value=1
    ```

1. クラスターの確認

    ```
    kubectl get node
    NAME                           STATUS   ROLES    AGE   VERSION
    ip-10-1-10-73.ec2.internal     Ready    <none>   32m   v1.19.6-eks-49a6c0
    ```

---


### デフォルトから変更する場合のメモ
1. 
1. 
1. 
1. 
1. 
1. 
1. 
1. 
1. 


