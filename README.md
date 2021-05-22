# memo-create-eks-env 　
※ EKSの検証環境を構築するメモです。

## 事前インストール
※Windowsで作業環境作ると大変  
※作業用のWorkstation環境を一時的に作る場合は[ここから](https://github.com/yamashin55/aws-workstation)作成できる  

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
    vi admin.auto.tfvars
    ```
    - resourceOwner：全部じゃないけど作成されるオブジェクトのOwnerタグにつける名前
    - awsRegion：リージョン指定
    - awsAz1：アベイラビリティゾーン１指定
    - awsAz2：アベイラビリティゾーン２指定
    - sshPublicKey：Workstationマシン作成されてそれに紐づけるキーペア（作成しない予定だけど一応記載）
    ```
    adminAccountName = "zadmin"  
    resourceOwner    = "syamada"  
    awsRegion        = "us-east-1"  
    awsAz1           = "us-east-1a"  
    awsAz2           = "us-east-1f"  
    sshPublicKey     = "ssh-rsa AAAAB3NzaC1.........."  
    ```

1. AWS CLIを使うためのセットアップ
    ```
    aws configure
    ```
    例：
    ```
    AWS Access Key ID [None]     : AKIAIOSFODNN7EXAMPLE
    AWS Secret Access Key [None] : wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    Default region name [None]   : us-east-1
    Default output format [None] : json
    ```

1. 環境構築
    ```
    ./setup.sh
    ```
    ※デフォルトの作成オブジェクトの内容を変更する場合は↓「[デフォルトから変更する場合のメモ](#デフォルトから変更する場合のメモ)」で先に編集してからコマンド実行する  
    ※作成までおよそ20分程  
    ※エラーが出たら、   ```terraform init --upgrade``` コマンド打つ  

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
    - Auto Scaling Group

1. ECRをコンテナリポジトリとして利用するために作成されたECRを登録する。

    「us-west-2」と「ecrRepositoryURL」は環境に合わせて変更する。  

    $ aws ecr get-login-password --region ***us-east-1*** | docker login --username AWS --password-stdin ***ecrRepositoryURL***  

    ```
    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 673420076654.dkr.ecr.us-east-1.amazonaws.com/demo-cluster-ecr-2372788194
    ```

1. Kube-configをアップデート

    「us-west-2」と「kubernetesClusterName」は環境に合わせて変更する。  

    $ aws eks --region ***us-east-1*** update-kubeconfig --name ***kubernetesClusterName***

    ```
    aws eks --region us-east-1 update-kubeconfig --name demo-cluster-2372788194
    ```

1. ELBを利用できるようにパブリックサブネットに2つのタグを配置する

    「publicSubnetAZ1」　「publicSubnetAZ2」　「kubernetesClusterName」は自分のに合わせて変更する。  

    $ aws ec2 create-tags \
    --resources ***publicSubnetAZ1*** ***publicSubnetAZ2*** \  
    --tags Key=kubernetes.io/cluster/***kubernetesClusterName***,Value=shared Key=kubernetes.io/role/elb,Value=1


    ```
    aws ec2 create-tags --resources subnet-0706ed6f210c9c338 subnet-017d2335b672300bd --tags Key=kubernetes.io/cluster/demo-cluster-2372788194,Value=shared   Key=kubernetes.io/role/elb,Value=1
    ```

1. クラスターの確認

    ```
    kubectl get node

    NAME                           STATUS   ROLES    AGE   VERSION
    ip-10-1-10-73.ec2.internal     Ready    <none>   32m   v1.19.6-eks-49a6c0
    ```

1. 環境削除

    ```bash
    ./cleanup.sh
    ```

---

## **デフォルトから変更する場合のメモ**

#### *`variables.tf`*

- 作成されるオブジェクトのプレフィックス（demo-awsに変更）  
  ```
  # cloud
  variable "projectPrefix" {
    default = "demo-aws"
  }
  ```

- 作成するリージョンを指定（us-east-1に変更）
  ```
  variable "awsRegion" {
    default = "us-east-1"
  }
  ```

- EKS Cluster の名前（my-cluster を変更）
  ```
  variable "clusterName" {
    default     = "my-cluster"
    description = "eks cluster name"
  }
  ```

#### *`main.tf`*

- AZの指定　（awsAz1、awsAz2の行を追加）
  ```
  // Network
  module "aws_network" {
    source                  = "../../../../../../modules/aws/terraform/network/min"
    projectPrefix           = var.projectPrefix
    awsRegion               = var.awsRegion
    awsAz1                  = var.awsAz1
    awsAz2                  = var.awsAz2
    map_public_ip_on_launch = true
  }
  ```

- K8sのバージョン変更（cluster_version = 1.19に変更）  
- ワーカノード数(Auto Scalling Group)の変更（asg_min_size=2、asg_desired_capacity=2 の行を追加）  
  ※ 作成時にワーカノード2台できる

  ```
  module "eks" {
    source          = "terraform-aws-modules/eks/aws"
    cluster_name    = "${var.clusterName}-${random_id.randomString.dec}"
    cluster_version = "1.19"
    subnets         = [module.aws_network.subnetsAz2["public"], module.aws_network.subnetsAz1["public"]]
    vpc_id          = module.aws_network.vpcs["main"]
    worker_groups = [
      {
        instance_type        = "t3.xlarge"
        asg_max_size         = 4
        asg_min_size         = 2
        asg_desired_capacity = 2
        root_volume_type     = "standard"
      }
    ]
    create_eks                           = true
    manage_aws_auth                      = true
    write_kubeconfig                     = true
    cluster_endpoint_private_access      = false
    cluster_endpoint_public_access       = true
    cluster_endpoint_public_access_cidrs = [var.adminSourceCidr]
    config_output_path                   = "${path.module}/cluster-config"
  }
  ```

- Linuxワークステーション作成しないように「jumphost」と「module "jumphost」全体をコメントアウト  
  ※このTerraformだとLinuxマシン1台が作成されるので、コメントアウトして作成しないようにする
  ```
  // jumphost
  // resource "aws_key_pair" "deployer" {
  //   key_name   = "${var.adminAccountName}-${var.projectPrefix}"
  //   public_key = var.sshPublicKey
  // }

  // module "jumphost" {
  //   source               = "../../../../../../modules/aws/terraform/workstation"
  //   projectPrefix        = var.projectPrefix
  //   adminAccountName     = var.adminAccountName
  //   coderAccountPassword = random_password.password.result
  //   vpc                  = module.aws_network.vpcs["main"]
  //   keyName              = aws_key_pair.deployer.id
  //   mgmtSubnet           = module.aws_network.subnetsAz1["mgmt"]
  //   securityGroup        = aws_security_group.secGroupWorkstation.id
  // }
  ```


#### *`outputs.tf`*

- Linuxワークステーション(jumphost)をコメントアウトするとエラーになるので、アウトプットファイルからもコメントアウト
  ```
  // output "jumphostPublicIp" {
  //   value = module.jumphost.workspaceManagementAddress
  // }

  // output "coderAdminPassword" {
  //   value = random_password.password.result
  // }
  ```

#### *`~/f5-digital-customer-engagement-center/modules/aws/terraform/network/min/main.tf`*

- 2つ目のAZを認識するよう「......? var.awsAz**2**」に修正
  ```
  locals {
    awsAz1 = var.awsAz1 != null ? var.awsAz1 : data.aws_availability_zones.available.names[0]
    awsAz2 = var.awsAz2 != null ? var.awsAz2 : data.aws_availability_zones.available.names[1]
  }
  ```

 #### *`~/f5-digital-customer-engagement-center/modules/aws/terraform/network/min/variables.tf`* 
- サブネットが既存で利用しているものと被っている場合は変更  
  ※ ワーカノードは **vpcMainSubPubX** 上に作成される)
  ```
  variable "vpcMainCidr" {
    default = "10.1.0.0/16"
  }
  variable "vpcMainSubPubACidr" {
    default = "10.1.10.0/24"
  }
  variable "vpcMainSubPubBCidr" {
    default = "10.1.110.0/24"
  }
  variable "vpcMainSubMgmtACidr" {
    default = "10.1.1.0/24"
  }
  variable "vpcMainSubMgmtBCidr" {
    default = "10.1.101.0/24"
  }
  variable "vpcMainSubPrivACidr" {
    default = "10.1.20.0/24"
  }
  variable "vpcMainSubPrivBCidr" {
    default = "10.1.120.0/24"
  }
  ```