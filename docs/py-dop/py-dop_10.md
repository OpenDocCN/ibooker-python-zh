# 第十章。基础设施即代码

在我们拥有花哨的 DevOps 标题和工作描述之前，我们是卑微的系统管理员，或者简称为 sysadmins。那些是黑暗的，云计算之前的日子，当时我们不得不将我们的车的行李箱装满裸金属服务器，然后开车到一个机房设施，将服务器安装到机架上，连接它们，连接一个有轮子的监视器/键盘/鼠标，然后逐个设置它们。格里格仍然不敢想象他在机房度过的小时，灯光刺眼，空调冷得刺骨。我们必须成为 Bash 脚本的巫师，然后我们毕业到 Perl，我们中更幸运的人到了 Python。俗话说，2004 年左右的互联网是用胶带和泡泡糖粘在一起的。

在 2006 年到 2007 年期间的某个时候，我们发现了亚马逊 EC2 实例的神奇世界。我们能够通过一个简单的点-and-click 接口或通过命令行工具来创建服务器。不再需要开车去机房，不再需要堆叠和连接裸金属服务器。我们可以疯狂地一次性启动 10 个 EC2 实例。甚至 20！甚至 100！天空是极限。然而，我们很快发现，手动连接到每个 EC2 实例，然后在每个实例上单独设置我们的应用程序不会扩展。创建实例本身相当容易。困难的是安装我们应用程序所需的软件包，添加正确的用户，确保文件权限正确，最后安装和配置我们的应用程序。为了解决这个问题，第一代基础架构自动化软件诞生了，“配置管理”工具代表着。Puppet 是第一个知名的配置管理工具，于 2005 年发布，早于 Amazon EC2 的发布。在 Puppet 推出后不久推出的其他类似工具包括 2008 年的 Chef，2011 年的 SaltStack，以及 2012 年的 Ansible。

到了 2009 年，世界准备迎来一个新术语的到来：DevOps。到今天为止，DevOps 有着竞争激烈的定义。有趣的是，它诞生在基础设施软件自动化的动荡早期。虽然 DevOps 中有重要的人和文化方面，但在这一章中有一件事是突出的：即自动化基础架构和应用程序的配置、部署和部署能力。

到了 2011 年，要跟踪亚马逊网络服务（AWS）套件中所有的服务变得越来越困难。云比起原始计算能力（Amazon EC2）和对象存储（Amazon S3）要复杂得多。应用程序开始依赖于相互交互的多个服务，并且需要工具来帮助自动化这些服务的配置。亚马逊没有等待太久就填补了这个需求，2011 年它开始提供这样的工具：AWS CloudFormation。这是我们真正能够通过代码描述基础设施的一个重要时刻之一。CloudFormation 为基础设施即代码（IaC）工具的新一代打开了大门，这些工具操作的是云基础设施层，低于第一代配置管理工具所提供的层次。

到了 2014 年，AWS 推出了数十项服务。那一年，另一个在 IaC 领域中重要的工具诞生了：HashiCorp 的 Terraform。时至今日，CloudFormation 和 Terraform 仍然是最常用的 IaC 工具。

在 IaC 和 DevOps 领域的另一个重要进展发生在 2013 年末到 2014 年初之间：Docker 的发布，它成为容器技术的代名词。尽管容器技术已经存在多年，但 Docker 为此带来的巨大好处在于，它将 Linux 容器和 cgroups 等技术包装成易于使用的 API 和命令行界面（CLI）工具集，大大降低了希望将其应用程序打包成容器并在任何 Docker 运行的地方部署和运行的人们的准入门槛。容器技术和容器编排平台在第十一章和第十二章中有详细讨论。

Docker 的使用率和影响力急剧上升，损害了第一代配置管理工具（Puppet、Chef、Ansible、SaltStack）的流行度。这些工具背后的公司目前正陷入困境，并都在试图通过重塑自身以适应云环境来保持活力和时效性。在 Docker 出现之前，您会使用诸如 CloudFormation 或 Terraform 等 IaC 工具来配置应用程序的基础设施，然后使用配置管理工具（如 Puppet、Chef、Ansible 或 SaltStack）部署应用程序本身（代码和配置）。Docker 突然间使得这些配置管理工具变得过时，因为它提供了一种方式，您可以将应用程序（代码+配置）打包到一个 Docker 容器中，然后在由 IaC 工具配置的基础设施内运行。

# 基础设施自动化工具分类

快进到 2020 年，作为一名 DevOps 从业者，在面对众多基础设施自动化工具时很容易感到迷失。

区分 IaC 工具的一种方式是看它们运行的层级。CloudFormation 和 Terraform 等工具运行在云基础设施层。它们允许你提供云资源，如计算、存储和网络，以及各种服务，如数据库、消息队列、数据分析等。配置管理工具如 Puppet、Chef、Ansible 和 SaltStack 通常在应用程序层操作，确保为你的应用程序安装所有所需的包，并确保应用程序本身配置正确（尽管这些工具中许多也有可以提供云资源的模块）。Docker 也在应用程序层操作。

另一种比较 IaC 工具的方法是将它们分为声明式和命令式两类。你可以用声明式方式告诉自动化工具要做什么，描述你想要实现的系统状态。Puppet、CloudFormation 和 Terraform 采用声明式方式操作。或者，你可以使用程序化或命令式方式使用自动化工具，指定工具需要执行的确切步骤来实现所需的系统状态。Chef 和 Ansible 采用命令式方式操作。SaltStack 可以同时采用声明式和命令式方式操作。

让我们把系统的期望状态看作建筑物（比如体育场）的建造蓝图。你可以使用 Chef 和 Ansible 这样的程序化工具，逐节、逐行地在每个部分内部建造体育场。你需要跟踪体育场的状态和建造进度。使用 Puppet、CloudFormation 和 Terraform 这样的声明式工具，你首先组装体育场的蓝图。然后工具确保建造达到蓝图中描述的状态。

鉴于本章的标题，我们将把剩下的讨论集中在 IaC 工具上，可以进一步按多个维度进行分类。

一个维度是指定系统期望状态的方式。在 CloudFormation 中，你可以使用 JSON 或 YAML 语法，而在 Terraform 中，你可以使用 HashiCorp Configuration Language (HCL) 语法。相比之下，Pulumi 和 AWS Cloud Development Kit (CDK) 允许你使用真正的编程语言，包括 Python，来指定系统的期望状态。

另一个维度是每个工具支持的云提供商。由于 CloudFormation 是亚马逊的服务，因此它专注于 AWS（尽管在使用自定义资源功能时可以定义非 AWS 资源）。AWS CDK 也是如此。相比之下，Terraform 支持许多云提供商，Pulumi 也是如此。

由于这是一本关于 Python 的书，我们想提到一个名为 [troposphere](https://oreil.ly/Zdid-) 的工具，它允许你使用 Python 代码指定 CloudFormation 堆栈模板，然后将其导出为 JSON 或 YAML。Troposphere 只负责生成堆栈模板，这意味着你需要使用 CloudFormation 进行堆栈的配置。另一个同样使用 Python 并值得一提的工具是 [stacker](https://oreil.ly/gBF_N)，它在底层使用 troposphere，但它还可以配置生成的 CloudFormation 堆栈模板。

本章的其余部分展示了两个自动化工具 Terraform 和 Pulumi 的实际操作，它们分别应用于一个常见的场景，即在 Amazon S3 中部署静态网站，该网站由 Amazon CloudFront CDN 托管，并通过 AWS 证书管理器（ACM）服务提供 SSL 证书保护。

###### 注意

在以下示例中使用的某些命令会生成大量输出。除非这些输出对理解命令至关重要，否则我们将省略大部分输出行以节省资源，并帮助你更好地专注于文本内容。

# 手动配置

我们首先通过 AWS 的 Web 控制台手动完成了一系列操作。没有什么比亲自体验手动操作的痛苦更能让你更好地享受自动化繁琐工作的成果了！

我们首先按照 AWS [S3 托管网站](https://oreil.ly/kdv8T) 的文档进行操作。

我们已经在 Namecheap 购买了一个域名：devops4all.dev。我们为该域名在 Amazon Route 53 中创建了托管区域，并将该域名在 Namecheap 中的名称服务器指向处理托管域名的 AWS DNS 服务器。

我们创建了两个 S3 存储桶，一个用于站点的根 URL（devops4all.dev），另一个用于 www URL（www.devops4all.dev）。我们的想法是将对 www 的请求重定向到根 URL。我们还按照指南配置了这些存储桶，使其支持静态网站托管，并设置了适当的权限。我们上传了一个 *index.html* 文件和一张 JPG 图片到根 S3 存储桶。

接下来的步骤是为处理根域名（devops4all.dev）及其任何子域名（*.devops4all.dev）的 SSL 证书进行配置。我们使用了添加到 Route 53 托管区域的 DNS 记录进行验证。

###### 注意

ACM 证书需要在 us-east-1 AWS 区域进行配置，以便在 CloudFront 中使用。

然后我们创建了一个 AWS CloudFront CDN 分发，指向根 S3 存储桶，并使用了前面配置的 ACM 证书。我们指定 HTTP 请求应重定向到 HTTPS。分发部署完成后（大约需要 15 分钟），我们添加了 Route 53 记录，将根域名和 www 域名作为类型为别名的 A 记录，指向 CloudFront 分发终端节点的 DNS 名称。

在完成本练习时，我们能够访问 *http://devops4all.dev*，自动重定向到 *https://devops4all.dev*，并看到我们上传的图片显示在站点首页上。我们还尝试访问 *http://www.devops4all.dev*，并被重定向到 *https://devops4all.dev*。

我们手动创建所有提到的 AWS 资源大约花了 30 分钟。此外，我们还花了 15 分钟等待 CloudFront 分发的传播，总共是 45 分钟。请注意，我们之前已经做过这些，所以我们几乎完全知道该怎么做，只需要最少量地参考 AWS 指南。

###### 注意

值得一提的是，如今很容易配置免费的 SSL 证书。早已不再需要等待 SSL 证书提供商审批您的请求，并提交证明您的公司存在的时间，只需使用 AWS ACM 或 Let’s Encrypt，2020 年再也没有不应该在站点的所有页面上启用 SSL 的借口。

# 使用 Terraform 进行自动化基础设施配置

我们决定使用 Terraform 作为首选的 IaC 工具来自动化这些任务，尽管 Terraform 与 Python 没有直接关系。它有几个优点，如成熟性、强大的生态系统和多云供应商。

编写 Terraform 代码的推荐方式是使用模块，这些模块是 Terraform 配置代码的可重用组件。HashiCorp 托管的 Terraform 模块的 [注册表](https://registry.terraform.io) 是一个通用的地方，您可以搜索用于配置所需资源的现成模块。在本示例中，我们将编写自己的模块。

此处使用的 Terraform 版本是 0.12.1，在撰写本文时是最新版本。在 Mac 上通过 `brew` 安装它：

```py
$ brew install terraform
```

## 配置一个 S3 存储桶

创建一个 *modules* 目录，在其下创建一个 *s3* 目录，其中包含三个文件：*main.tf*、*variables.tf* 和 *outputs.tf*。*s3* 目录中的 *main.tf* 文件告诉 Terraform 创建一个具有特定策略的 S3 存储桶。它使用一个名为 `domain_name` 的变量，在 *variables.tf* 中声明，其值由调用此模块的用户传递给它。它输出 S3 存储桶的 DNS 端点，其他模块将使用它作为输入变量。

这里是 *modules/s3* 中的三个文件：

```py
$ cat modules/s3/main.tf
resource "aws_s3_bucket" "www" {
  bucket = "www.${var.domain_name}"
  acl = "public-read"
  policy = <<POLICY
{
  "Version":"2012-10-17",
  "Statement":[
    {
      "Sid":"AddPerm",
      "Effect":"Allow",
      "Principal": "*",
      "Action":["s3:GetObject"],
      "Resource":["arn:aws:s3:::www.${var.domain_name}/*"]
    }
  ]
}
POLICY

  website {
    index_document = "index.html"
  }
}

$ cat modules/s3/variables.tf
variable "domain_name" {}

$ cat modules/s3/outputs.tf
output "s3_www_website_endpoint" {
  value = "${aws_s3_bucket.www.website_endpoint}"
}
```

###### 注意

上述 `aws_s3_bucket` 资源的 `policy` 属性是允许公共访问该存储桶的 S3 存储桶策略的一个示例。如果您在 IaC 环境中使用 S3 存储桶，请熟悉 [官方 AWS 存储桶和用户策略文档](https://oreil.ly/QtTYd)。

将所有模块绑定在一起的主要 Terraform 脚本位于当前目录中的名为 *main.tf* 的文件中：

```py
$ cat main.tf
provider "aws" {
  region = "${var.aws_region}"
}

module "s3" {
  source = "./modules/s3"
  domain_name = "${var.domain_name}"
}
```

它引用了一个定义在名为 *variables.tf* 的单独文件中的变量：

```py
$ cat variables.tf
variable "aws_region" {
  default = "us-east-1"
}

variable "domain_name" {
  default = "devops4all.dev"
}
```

这是当前目录树的情况：

```py
|____main.tf
|____variables.tf
|____modules
| |____s3
| | |____outputs.tf
| | |____main.tf
| | |____variables.tf
```

运行 Terraform 的第一步是调用 `terraform init` 命令，它将读取主文件引用的任何模块的内容。

接下来的步骤是运行 `terraform plan` 命令，它创建了前面讨论中提到的蓝图。

要创建计划中指定的资源，请运行 `terraform apply` 命令：

```py
$ terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.s3.aws_s3_bucket.www will be created
  + resource "aws_s3_bucket" "www" {
      + acceleration_status = (known after apply)
      + acl  = "public-read"
      + arn  = (known after apply)
      + bucket  = "www.devops4all.dev"
      + bucket_domain_name  = (known after apply)
      + bucket_regional_domain_name = (known after apply)
      + force_destroy = false
      + hosted_zone_id= (known after apply)
      + id= (known after apply)
      + policy  = jsonencode(
            {
              + Statement = [
                  + {
                      + Action = [
                          + "s3:GetObject",
                        ]
                      + Effect = "Allow"
                      + Principal = "*"
                      + Resource  = [
                          + "arn:aws:s3:::www.devops4all.dev/*",
                        ]
                      + Sid = "AddPerm"
                    },
                ]
              + Version= "2012-10-17"
            }
        )
      + region  = (known after apply)
      + request_payer = (known after apply)
      + website_domain= (known after apply)
      + website_endpoint = (known after apply)

      + versioning {
          + enabled = (known after apply)
          + mfa_delete = (known after apply)
        }

      + website {
          + index_document = "index.html"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

module.s3.aws_s3_bucket.www: Creating...
module.s3.aws_s3_bucket.www: Creation complete after 7s [www.devops4all.dev]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

此时，请检查是否使用 AWS Web 控制台 UI 创建了 S3 存储桶。

## 使用 AWS ACM 配置 SSL 证书

下一个模块是为使用 AWS 证书管理器服务进行 SSL 证书配置而创建的。创建一个名为 *modules/acm* 的目录，其中包含三个文件：*main.tf*、*variables.tf* 和 *outputs.tf*。*acm* 目录中的 *main.tf* 文件告诉 Terraform 使用 DNS 作为验证方法创建 ACM SSL 证书。它使用一个名为 `domain_name` 的变量，该变量在 *variables.tf* 中声明，并由调用此模块的调用者传递其值。它输出证书的 ARN 标识符，该标识符将被其他模块用作输入变量。

```py
$ cat modules/acm/main.tf
resource "aws_acm_certificate" "certificate" {
  domain_name = "*.${var.domain_name}"
  validation_method = "DNS"
  subject_alternative_names = ["*.${var.domain_name}"]
}

$ cat modules/acm/variables.tf
variable "domain_name" {
}

$ cat modules/acm/outputs.tf
output "certificate_arn" {
  value = "${aws_acm_certificate.certificate.arn}"
}
```

在主 Terraform 文件中添加对新的 `acm` 模块的引用：

```py
$ cat main.tf
provider "aws" {
  region = "${var.aws_region}"
}

module "s3" {
  source = "./modules/s3"
  domain_name = "${var.domain_name}"
}

module "acm" {
  source = "./modules/acm"
  domain_name = "${var.domain_name}"
}
```

接下来的三个步骤与 S3 存储桶创建序列中的步骤相同：`terraform init`、`terraform plan` 和 `terraform apply`。

使用 AWS 控制台添加必要的 Route 53 记录以进行验证过程。证书通常会在几分钟内验证和发布。

## 配置 Amazon CloudFront 分发

下一个模块是为创建 Amazon CloudFront 分发而创建的。创建一个名为 *modules/cloudfront* 的目录，其中包含三个文件：*main.tf*、*variables.tf* 和 *outputs.tf*。*cloudfront* 目录中的 *main.tf* 文件告诉 Terraform 创建 CloudFront 分发资源。它使用在 *variables.tf* 中声明的多个变量，这些变量的值由调用此模块的调用者传递。它输出 CloudFront 端点的 DNS 域名和 CloudFront 分发的托管 Route 53 区域 ID，这些将作为其他模块的输入变量使用：

```py
$ cat modules/cloudfront/main.tf
resource "aws_cloudfront_distribution" "www_distribution" {
  origin {
    custom_origin_config {
      // These are all the defaults.
      http_port= "80"
      https_port  = "443"
      origin_protocol_policy = "http-only"
      origin_ssl_protocols= ["TLSv1", "TLSv1.1", "TLSv1.2"]
    }

    domain_name = "${var.s3_www_website_endpoint}"
    origin_id= "www.${var.domain_name}"
  }

  enabled  = true
  default_root_object = "index.html"

  default_cache_behavior {
    viewer_protocol_policy = "redirect-to-https"
    compress = true
    allowed_methods= ["GET", "HEAD"]
    cached_methods = ["GET", "HEAD"]
    target_origin_id = "www.${var.domain_name}"
    min_ttl  = 0
    default_ttl = 86400
    max_ttl  = 31536000

    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }
  }

  aliases = ["www.${var.domain_name}"]

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    acm_certificate_arn = "${var.acm_certificate_arn}"
    ssl_support_method  = "sni-only"
  }
}

$ cat modules/cloudfront/variables.tf
variable "domain_name" {}
variable "acm_certificate_arn" {}
variable "s3_www_website_endpoint" {}

$ cat modules/cloudfront/outputs.tf
output "domain_name" {
  value = "${aws_cloudfront_distribution.www_distribution.domain_name}"
}

output "hosted_zone_id" {
  value = "${aws_cloudfront_distribution.www_distribution.hosted_zone_id}"
}
```

在主 Terraform 文件中添加对 `cloudfront` 模块的引用。将 `s3_www_website_endpoint` 和 `acm_certificate_arn` 作为输入变量传递给 `cloudfront` 模块。它们的值从其他模块 `s3` 和 `acm` 的输出中获取。

###### 注意

ARN 代表 Amazon 资源名称。它是一个字符串，用于唯一标识给定的 AWS 资源。当您使用在 AWS 内运行的 IaC 工具时，会看到许多生成的 ARN 值作为变量传递和传递。

```py
$ cat main.tf
provider "aws" {
  region = "${var.aws_region}"
}

module "s3" {
  source = "./modules/s3"
  domain_name = "${var.domain_name}"
}

module "acm" {
  source = "./modules/acm"
  domain_name = "${var.domain_name}"
}

module "cloudfront" {
  source = "./modules/cloudfront"
  domain_name = "${var.domain_name}"
  s3_www_website_endpoint = "${module.s3.s3_www_website_endpoint}"
  acm_certificate_arn = "${module.acm.certificate_arn}"
}
```

接下来的三个步骤是使用 Terraform 进行资源配置的常规步骤：`terraform init`、`terraform plan` 和 `terraform apply`。

在这种情况下，`terraform apply` 步骤耗时约 23 分钟。创建 Amazon CloudFront 分发是 AWS 中最耗时的操作之一，因为该分发在幕后由 Amazon 在全球范围内部署。

## 配置 Route 53 DNS 记录

下一个模块是为站点 www.devops4all.dev 的主域创建 Route 53 DNS 记录。创建一个名为 *modules/route53* 的目录，并包含两个文件：*main.tf* 和 *variables.tf*。*route53* 目录中的 *main.tf* 文件告诉 Terraform 创建一个类型为 `A` 的 Route 53 DNS 记录，作为 CloudFront 终端节点的 DNS 名称的别名。它使用在 *variables.tf* 中声明的几个变量，并通过调用此模块的调用者传递给它的值：

```py
$ cat modules/route53/main.tf
resource "aws_route53_record" "www" {
  zone_id = "${var.zone_id}"
  name = "www.${var.domain_name}"
  type = "A"

  alias {
    name  = "${var.cloudfront_domain_name}"
    zone_id  = "${var.cloudfront_zone_id}"
    evaluate_target_health = false
  }
}

$ cat modules/route53/variables.tf
variable "domain_name" {}
variable "zone_id" {}
variable "cloudfront_domain_name" {}
variable "cloudfront_zone_id" {}
```

在 *main.tf* Terraform 文件中添加对 `route53` 模块的引用。将 `zone_id`、`cloudfront_domain_name` 和 `cloudfront_zone_id` 作为输入变量传递给 `route53` 模块。`zone_id` 的值在当前目录的 *variables.tf* 中声明，而其他值则从 `cloudfront` 模块的输出中检索：

```py
$ cat main.tf
provider "aws" {
  region = "${var.aws_region}"
}

module "s3" {
  source = "./modules/s3"
  domain_name = "${var.domain_name}"
}

module "acm" {
  source = "./modules/acm"
  domain_name = "${var.domain_name}"
}

module "cloudfront" {
  source = "./modules/cloudfront"
  domain_name = "${var.domain_name}"
  s3_www_website_endpoint = "${module.s3.s3_www_website_endpoint}"
  acm_certificate_arn = "${module.acm.certificate_arn}"
}

module "route53" {
  source = "./modules/route53"
  domain_name = "${var.domain_name}"
  zone_id = "${var.zone_id}"
  cloudfront_domain_name = "${module.cloudfront.domain_name}"
  cloudfront_zone_id = "${module.cloudfront.hosted_zone_id}"
}

$ cat variables.tf
variable "aws_region" {
  default = "us-east-1"
}

variable "domain_name" {
  default = "devops4all.dev"
}

variable "zone_id" {
  default = "ZWX18ZIVHAA5O"
}
```

接下来的三个步骤，现在对您来说应该非常熟悉了，是使用 Terraform 配置资源的：`terraform init`、`terraform plan` 和 `terraform apply`。

## 将静态文件复制到 S3

为了测试从头到尾创建静态网站的配置，创建一个名为 *index.html* 的简单文件，其中包含一个 JPEG 图像，并将这两个文件复制到之前使用 Terraform 配置的 S3 存储桶。确保 `AWS_PROFILE` 环境变量已设置为 *~/.aws/credentials* 文件中已存在的正确值：

```py
$ echo $AWS_PROFILE
gheorghiu-net
$ aws s3 cp static_files/index.html s3://www.devops4all.dev/index.html
upload: static_files/index.html to s3://www.devops4all.dev/index.html
$ aws s3 cp static_files/devops4all.jpg s3://www.devops4all.dev/devops4all.jpg
upload: static_files/devops4all.jpg to s3://www.devops4all.dev/devops4all.jpg
```

访问 [*https://www.devops4all.dev/*](https://www.devops4all.dev/) 并验证您是否可以看到已上传的 JPG 图像。

## 使用 Terraform 删除所有已配置的 AWS 资源

每当您配置云资源时，都需要注意与其相关的费用。很容易忘记它们，您可能会在月底收到意外的 AWS 账单。确保删除上面配置的所有资源。通过运行 `terraform destroy` 命令来删除这些资源。还要注意，在运行 `terraform destroy` 之前需要删除 S3 存储桶的内容，因为 Terraform 不会删除非空桶的内容。

###### 注意

在运行 `terraform destroy` 命令之前，请确保您不会删除仍可能在生产环境中使用的资源！

# 使用 Pulumi 自动化基础设施配置

当涉及到 IaC 工具时，Pulumi 是新秀之一。关键词是 *new*，这意味着它在某些方面仍然有些粗糙，特别是在 Python 支持方面。

Pulumi 允许您通过告诉它使用真正的编程语言来提供所需的基础设施状态，来指定您的基础设施的期望状态。TypeScript 是 Pulumi 支持的第一种语言，但现在也支持 Go 和 Python。

理解在 Python 中使用 Pulumi 编写基础设施自动化代码与使用 AWS 自动化库（如 Boto）之间的区别至关重要。

使用 Pulumi，您的 Python 代码描述了要部署的资源。实际上，您正在创建本章开头讨论的蓝图或状态。这使得 Pulumi 类似于 Terraform，但其主要区别在于 Pulumi 允许您充分利用像 Python 这样的编程语言的全部功能，例如编写函数、循环、使用变量等。您不受 Terraform 的 HCL 等标记语言的限制。Pulumi 结合了声明式方法的强大之处（描述所需的最终状态）与真正编程语言的强大之处。

使用诸如 Boto 之类的 AWS 自动化库，您可以通过编写的代码描述和提供单个 AWS 资源。没有整体蓝图或状态。您需要自行跟踪已提供的资源，并协调其创建和移除。这是自动化工具的命令式或过程化方法。您仍然可以利用编写 Python 代码的优势。

要开始使用 Pulumi，请在他们的网站 pulumi.io 上创建一个免费帐户。然后，您可以在本地计算机上安装`pulumi`命令行工具。在 Macintosh 上，请使用 Homebrew 安装`pulumi`。

本地运行的第一个命令是`pulumi login`：

```py
$ pulumi login
Logged into pulumi.com as griggheo (https://app.pulumi.com/griggheo)
```

## 创建 AWS 的新 Pulumi Python 项目

创建一个名为*proj1*的目录，在该目录中运行`pulumi new`，并选择`aws-python`模板。在项目创建的过程中，`pulumi`会要求您为堆栈命名。称其为`staging`：

```py
$ mkdir proj1
$ cd proj1
$ pulumi new
Please choose a template: aws-python        A minimal AWS Python Pulumi program
This command will walk you through creating a new Pulumi project.

Enter a value or leave blank to accept the (default), and press <ENTER>.
Press ^C at any time to quit.

project name: (proj1)
project description: (A minimal AWS Python Pulumi program)
Created project 'proj1'

stack name: (dev) staging
Created stack 'staging'

aws:region: The AWS region to deploy into: (us-east-1)
Saved config

Your new project is ready to go!
To perform an initial deployment, run the following commands:

   1\. virtualenv -p python3 venv
   2\. source venv/bin/activate
   3\. pip3 install -r requirements.txt

Then, run 'pulumi up'
```

重要的是要理解 Pulumi 项目与 Pulumi 堆栈之间的区别。项目是您为指定系统所需状态编写的代码，即您希望 Pulumi 提供的资源。堆栈是项目的特定部署。例如，堆栈可以对应于开发、暂存或生产等环境。在接下来的示例中，我们将创建两个 Pulumi 堆栈，一个称为`staging`，对应于暂存环境，以及稍后的另一个称为`prod`，对应于生产环境。

这里是由`pulumi new`命令自动生成的文件，作为`aws-python`模板的一部分：

```py
$ ls -la
total 40
drwxr-xr-x   7 ggheo  staff  224 Jun 13 21:43 .
drwxr-xr-x  11 ggheo  staff  352 Jun 13 21:42 ..
-rw-------   1 ggheo  staff   12 Jun 13 21:43 .gitignore
-rw-r--r--   1 ggheo  staff   32 Jun 13 21:43 Pulumi.staging.yaml
-rw-------   1 ggheo  staff   77 Jun 13 21:43 Pulumi.yaml
-rw-------   1 ggheo  staff  184 Jun 13 21:43 __main__.py
-rw-------   1 ggheo  staff   34 Jun 13 21:43 requirements.txt
```

按照`pulumi new`的输出指示安装`virtualenv`，然后创建新的`virtualenv`环境并安装*requirements.txt*中指定的库：

```py
$ pip3 install virtualenv
$ virtualenv -p python3 venv
$ source venv/bin/activate
(venv) pip3 install -r requirements.txt
```

###### 注意

在使用`pulumi up`之前，确保您正在使用预期目标的 AWS 帐户来配置任何 AWS 资源。指定所需 AWS 帐户的一种方法是在当前 shell 中设置`AWS_PROFILE`环境变量。在我们的情况下，本地*~/.aws/credentials*文件中已设置了名为`gheorghiu-net`的 AWS 配置文件。

```py
(venv) export AWS_PROFILE=gheorghiu-net
```

由 Pulumi 作为`aws-python`模板的一部分生成的*__main__.py*文件如下：

```py
$ cat __main__.py
import pulumi
from pulumi_aws import s3

# Create an AWS resource (S3 Bucket)
bucket = s3.Bucket('my-bucket')

# Export the name of the bucket
pulumi.export('bucket_name',  bucket.id)
```

本地克隆[Pulumi 示例 GitHub 存储库](https://oreil.ly/SIT-v)，然后将\_\_main\_\_.py 和*pulumi-examples/aws-py-s3-folder*中的 www 目录复制到当前目录中。

这里是当前目录中的新\_\_main\_\_.py 文件：

```py
$ cat __main__.py
import json
import mimetypes
import os

from pulumi import export, FileAsset
from pulumi_aws import s3

web_bucket = s3.Bucket('s3-website-bucket', website={
    "index_document": "index.html"
})

content_dir = "www"
for file in os.listdir(content_dir):
    filepath = os.path.join(content_dir, file)
    mime_type, _ = mimetypes.guess_type(filepath)
    obj = s3.BucketObject(file,
        bucket=web_bucket.id,
        source=FileAsset(filepath),
        content_type=mime_type)

def public_read_policy_for_bucket(bucket_name):
    return json.dumps({
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                f"arn:aws:s3:::{bucket_name}/*",
            ]
        }]
    })

bucket_name = web_bucket.id
bucket_policy = s3.BucketPolicy("bucket-policy",
    bucket=bucket_name,
    policy=bucket_name.apply(public_read_policy_for_bucket))

# Export the name of the bucket
export('bucket_name',  web_bucket.id)
export('website_url', web_bucket.website_endpoint)
```

请注意 Python 变量`content_dir`和`bucket_name`的使用，`for`循环的使用，以及正常 Python 函数`public_read_policy_for_bucket`的使用。能够在 IaC 程序中使用正常的 Python 结构感到耳目一新！

现在是运行`pulumi up`以提供在\_\_main\_\_.py 中指定的资源的时候了。该命令将展示将要创建的所有资源。将当前选择移动到`yes`将启动配置过程：

```py
(venv) pulumi up
Previewing update (staging):

     Type                    Name               Plan
 +   pulumi:pulumi:Stack     proj1-staging      create
 +   ├─ aws:s3:Bucket        s3-website-bucket  create
 +   ├─ aws:s3:BucketObject  favicon.png        create
 +   ├─ aws:s3:BucketPolicy  bucket-policy      create
 +   ├─ aws:s3:BucketObject  python.png         create
 +   └─ aws:s3:BucketObject  index.html         create

Resources:
    + 6 to create

Do you want to perform this update? yes
Updating (staging):

     Type                    Name               Status
 +   pulumi:pulumi:Stack     proj1-staging      created
 +   ├─ aws:s3:Bucket        s3-website-bucket  created
 +   ├─ aws:s3:BucketObject  index.html         created
 +   ├─ aws:s3:BucketObject  python.png         created
 +   ├─ aws:s3:BucketObject  favicon.png        created
 +   └─ aws:s3:BucketPolicy  bucket-policy      created

Outputs:
    bucket_name: "s3-website-bucket-8e08f8f"
    website_url: "s3-website-bucket-8e08f8f.s3-website-us-east-1.amazonaws.com"

Resources:
    + 6 created

Duration: 14s
```

检查现有的 Pulumi 堆栈：

```py
(venv) pulumi stack ls
NAME      LAST UPDATE    RESOURCE COUNT  URL
staging*  2 minutes ago  7        https://app.pulumi.com/griggheo/proj1/staging

(venv) pulumi stack
Current stack is staging:
    Owner: griggheo
    Last updated: 3 minutes ago (2019-06-13 22:05:38.088773 -0700 PDT)
    Pulumi version: v0.17.16
Current stack resources (7):
    TYPE                              NAME
    pulumi:pulumi:Stack               proj1-staging
    pulumi:providers:aws              default
    aws:s3/bucket:Bucket              s3-website-bucket
    aws:s3/bucketPolicy:BucketPolicy  bucket-policy
    aws:s3/bucketObject:BucketObject  index.html
    aws:s3/bucketObject:BucketObject  favicon.png
    aws:s3/bucketObject:BucketObject  python.png
```

检查当前堆栈的输出：

```py
(venv) pulumi stack output
Current stack outputs (2):
    OUTPUT       VALUE
    bucket_name  s3-website-bucket-8e08f8f
    website_url  s3-website-bucket-8e08f8f.s3-website-us-east-1.amazonaws.com
```

访问`website_url`输出中指定的 URL（[*http://s3-website-bucket-8e08f8f.s3-website-us-east-1.amazonaws.com*](http://s3-website-bucket-8e08f8f.s3-website-us-east-1.amazonaws.com)），确保您可以看到静态站点。

在接下来的章节中，将通过指定更多的 AWS 资源来增强 Pulumi 项目。目标是与使用 Terraform 配置的资源保持一致：一个 ACM SSL 证书，一个 CloudFront 分发和一个用于站点 URL 的 Route 53 DNS 记录。

## 为 Staging 堆栈创建配置值

当前堆栈为`staging`。将现有的 www 目录重命名为 www-staging，然后使用`pulumi config set`命令为当前`staging`堆栈指定两个配置值：`domain_name`和`local_webdir`。

###### 提示

有关 Pulumi 如何管理配置值和密钥的详细信息，请参阅[Pulumi 参考文档](https://oreil.ly/D_Cy5)。

```py
(venv) mv www www-staging
(venv) pulumi config set local_webdir www-staging
(venv) pulumi config set domain_name staging.devops4all.dev
```

要检查当前堆栈的现有配置值，请运行：

```py
(venv) pulumi config
KEY           VALUE
aws:region    us-east-1
domain_name   staging.devops4all.dev
local_webdir  www-staging
```

配置值设置完毕后，将它们用于 Pulumi 代码中：

```py
import pulumi

config = pulumi.Config('proj1')  # proj1 is project name defined in Pulumi.yaml

content_dir = config.require('local_webdir')
domain_name = config.require('domain_name')
```

现在配置值已经就位；接下来我们将使用 AWS 证书管理器服务来提供 SSL 证书。

## 配置 ACM SSL 证书

大约在这一点上，当涉及到其 Python SDK 时，Pulumi 开始显露出其不足之处。仅仅阅读 Pulumi Python SDK 参考中的[`acm`](https://oreil.ly/Niwaj)模块并不足以理解您在 Pulumi 程序中需要做的事情。

幸运的是，有许多 Pulumi TypeScript 示例可供您参考。一个展示我们用例的示例是[`aws-ts-static-website`](https://oreil.ly/7F39c)。

这是创建新 ACM 证书的 TypeScript 代码（来自[`index.ts`](https://oreil.ly/mlSr1)）：

```py
const certificate = new aws.acm.Certificate("certificate", {
    domainName: config.targetDomain,
    validationMethod: "DNS",
}, { provider: eastRegion });
```

这是我们编写的相应 Python 代码：

```py
from pulumi_aws import acm

cert = acm.Certificate('certificate', domain_name=domain_name,
    validation_method='DNS')
```

###### 提示

从 TypeScript 转换 Pulumi 代码到 Python 的一个经验法则是，在 TypeScript 中使用驼峰命名法的参数在 Python 中会变成蛇形命名法。正如您在前面的示例中看到的，`domainName`变成了`domain_name`，而`validationMethod`变成了`validation_method`。

我们的下一步是为 ACM SSL 证书配置 Route 53 区域，并在该区域中为 DNS 验证记录。

## 配置 Route 53 区域和 DNS 记录

如果您遵循 [Pulumi SDK reference for route53](https://oreil.ly/cU9Yj)，使用 Pulumi 配置新的 Route 53 区域非常容易。

```py
from pulumi_aws import route53

domain_name = config.require('domain_name')

# Split a domain name into its subdomain and parent domain names.
# e.g. "www.example.com" => "www", "example.com".
def get_domain_and_subdomain(domain):
  names = domain.split(".")
  if len(names) < 3:
    return('', domain)
  subdomain = names[0]
  parent_domain = ".".join(names[1:])
  return (subdomain, parent_domain)

(subdomain, parent_domain) = get_domain_and_subdomain(domain_name)
zone = route53.Zone("route53_zone", name=parent_domain)
```

前面的代码片段显示了如何使用常规 Python 函数将读取的配置值拆分为 `domain_name` 变量的两部分。如果 `domain_name` 是 `staging.devops4all.dev`，则函数将其拆分为 `subdomain` (`staging`) 和 `parent_domain` (`devops4all.dev`)。

`parent_domain` 变量然后作为 `zone` 对象的构造函数的参数使用，告诉 Pulumi 配置 `route53.Zone` 资源。

###### 注意

创建 Route 53 区域后，我们必须将 Namecheap 的域名服务器指向新区域的 DNS 记录中指定的域名服务器，以便该区域可以公开访问。

到目前为止，一切都很顺利。接下来的步骤是同时创建 ACM 证书和用于验证证书的 DNS 记录。

我们首先尝试通过将 camelCase 参数名转换为 snake_case 的经验法则来移植示例 TypeScript 代码。

TypeScript:

```py
    const certificateValidationDomain = new aws.route53.Record(
        `${config.targetDomain}-validation`, {
        name: certificate.domainValidationOptions[0].resourceRecordName,
        zoneId: hostedZoneId,
        type: certificate.domainValidationOptions[0].resourceRecordType,
        records: [certificate.domainValidationOptions[0].resourceRecordValue],
        ttl: tenMinutes,
    });
```

第一次尝试通过将 camelCase 转换为 snake_case 来将示例移植到 Python：

```py
cert = acm.Certificate('certificate',
    domain_name=domain_name, validation_method='DNS')

domain_validation_options = cert.domain_validation_options[0]

cert_validation_record = route53.Record(
  'cert-validation-record',
  name=domain_validation_options.resource_record_name,
  zone_id=zone.id,
  type=domain_validation_options.resource_record_type,
  records=[domain_validation_options.resource_record_value],
  ttl=600)
```

运气不佳。`pulumi up` 显示如下错误：

```py
AttributeError: 'dict' object has no attribute 'resource_record_name'
```

在这一点上，我们陷入困境，因为 Python SDK 文档没有包含这么详细的信息。我们不知道在 `domain_validation_options` 对象中需要指定哪些属性。

我们只能通过在 Pulumi 的导出列表中添加 `domain_validation_options` 对象来解决此问题，这些对象由 Pulumi 在 `pulumi up` 操作结束时打印出来。

```py
export('domain_validation_options', domain_validation_options)
```

`pulumi up` 的输出如下：

```py
+ domain_validation_options: {
  + domain_name        : "staging.devops4all.dev"
  + resourceRecordName : "_c5f82e0f032d0f4f6c7de17fc2c.staging.devops4all.dev."
  + resourceRecordType : "CNAME"
  + resourceRecordValue: "_08e3d475bf3aeda0c98.ltfvzjuylp.acm-validations.aws."
    }
```

终于找到了！原来 `domain_validation_options` 对象的属性仍然是驼峰命名法。

这是第二次成功移植到 Python 的尝试：

```py
cert_validation_record = route53.Record(
  'cert-validation-record',
  name=domain_validation_options['resourceRecordName'],
  zone_id=zone.id,
  type=domain_validation_options['resourceRecordType'],
  records=[domain_validation_options['resourceRecordValue']],
  ttl=600)
```

接下来，指定要配置的新类型资源：证书验证完成资源。这使得 `pulumi up` 操作等待 ACM 通过检查先前创建的 Route 53 验证记录来验证证书。

```py
cert_validation_completion = acm.CertificateValidation(
        'cert-validation-completion',
        certificate_arn=cert.arn,
        validation_record_fqdns=[cert_validation_dns_record.fqdn])

cert_arn = cert_validation_completion.certificate_arn
```

到此为止，您已经拥有了通过完全自动化的方式配置 ACM SSL 证书并通过 DNS 进行验证的方法。

接下来的步骤是在托管站点静态文件的 S3 存储桶前面配置 CloudFront 分发。

## 配置 CloudFront 分发

使用 Pulumi [`cloudfront` 模块](https://oreil.ly/4n98-) 的 SDK 参考来确定传递给 `cloudfront.Distribution` 的构造函数参数。检查 TypeScript 代码以了解这些参数的正确值。

这里是最终结果：

```py
log_bucket = s3.Bucket('cdn-log-bucket', acl='private')

cloudfront_distro = cloudfront.Distribution ( 'cloudfront-distro',
    enabled=True,
    aliases=[ domain_name ],
    origins=[
        {
          'originId': web_bucket.arn,
          'domainName': web_bucket.website_endpoint,
          'customOriginConfig': {
              'originProtocolPolicy': "http-only",
              'httpPort': 80,
              'httpsPort': 443,
              'originSslProtocols': ["TLSv1.2"],
            },
        },
    ],

    default_root_object="index.html",
    default_cache_behavior={
        'targetOriginId': web_bucket.arn,

        'viewerProtocolPolicy': "redirect-to-https",
        'allowedMethods': ["GET", "HEAD", "OPTIONS"],
        'cachedMethods': ["GET", "HEAD", "OPTIONS"],

        'forwardedValues': {
            'cookies': { 'forward': "none" },
            'queryString': False,
        },

        'minTtl': 0,
        'defaultTtl': 600,
        'maxTtl': 600,
    },
    price_class="PriceClass_100",
    custom_error_responses=[
        { 'errorCode': 404, 'responseCode': 404,
          'responsePagePath': "/404.html" },
    ],

    restrictions={
        'geoRestriction': {
            'restrictionType': "none",
        },
    },
    viewer_certificate={
        'acmCertificateArn': cert_arn,
        'sslSupportMethod': "sni-only",
    },
    logging_config={
        'bucket': log_bucket.bucket_domain_name,
        'includeCookies': False,
        'prefix': domain_name,
    })
```

运行 `pulumi up` 以配置 CloudFront 分发。

## 为站点 URL 配置 Route 53 DNS 记录

在端到端为 `staging` 栈配置资源的最后一步是将 `A` 类型的 DNS 记录作为 CloudFront 终端节点的域的别名指定。

```py
site_dns_record = route53.Record(
        'site-dns-record',
        name=subdomain,
        zone_id=zone.id,
        type="A",
        aliases=[
        {
            'name': cloudfront_distro.domain_name,
            'zoneId': cloudfront_distro.hosted_zone_id,
            'evaluateTargetHealth': True
        }
    ])
```

如常运行 `pulumi up`。

访问 [*https://staging.devops4all.dev*](https://staging.devops4all.dev)，查看上传到 S3 的文件。转到 AWS 控制台中的日志桶，并确保 CloudFront 日志在那里。

让我们看看如何将相同的 Pulumi 项目部署到一个新的环境，由新的 Pulumi 栈表示。

## 创建和部署新栈

我们决定修改 Pulumi 程序，使其不再创建新的 Route 53 区域，而是使用现有区域的区域 ID 作为配置值。

要创建 `prod` 栈，请使用命令 `pulumi stack init` 并将其名称指定为 `prod`：

```py
(venv) pulumi stack init
Please enter your desired stack name: prod
Created stack 'prod'
```

列出栈现在显示了两个栈，`staging` 和 `prod`，`prod` 栈旁边带有星号，表示 `prod` 是当前栈：

```py
(venv) pulumi stack ls
NAME     LAST UPDATE     RESOURCE COUNT  URL
prod*    n/a             n/a      https://app.pulumi.com/griggheo/proj1/prod
staging  14 minutes ago  14       https://app.pulumi.com/griggheo/proj1/staging
```

现在是为 `prod` 栈设置正确配置值的时候了。使用一个新的 `dns_zone_id` 配置值，设置为在 Pulumi 配置 `staging` 栈时已创建的区域的 ID：

```py
(venv) pulumi config set aws:region us-east-1
(venv) pulumi config set local_webdir www-prod
(venv) pulumi config set domain_name www.devops4all.dev
(venv) pulumi config set dns_zone_id Z2FTL2X8M0EBTW
```

更改代码以从配置中读取 `zone_id` 并且不创建 Route 53 区域对象。

运行 `pulumi up` 配置 AWS 资源：

```py
(venv) pulumi up
Previewing update (prod):

     Type                            Name               Plan
     pulumi:pulumi:Stack             proj1-prod
 +   ├─ aws:cloudfront:Distribution  cloudfront-distro  create
 +   └─ aws:route53:Record           site-dns-record    create

Resources:
    + 2 to create
    10 unchanged

Do you want to perform this update? yes
Updating (prod):

     Type                            Name               Status
     pulumi:pulumi:Stack             proj1-prod
 +   ├─ aws:cloudfront:Distribution  cloudfront-distro  created
 +   └─ aws:route53:Record           site-dns-record    created

Outputs:
+ cloudfront_domain: "d3uhgbdw67nmlc.cloudfront.net"
+ log_bucket_id    : "cdn-log-bucket-53d8ea3"
+ web_bucket_id    : "s3-website-bucket-cde"
+ website_url      : "s3-website-bucket-cde.s3-website-us-east-1.amazonaws.com"

Resources:
    + 2 created
    10 unchanged

Duration: 18m54s
```

成功！`prod` 栈已完全部署。

然而，此时 *www-prod* 目录中包含站点静态文件的内容与 *www-staging* 目录中的内容相同。

修改 *www-prod/index.html*，将“Hello, S3!”改为“Hello, S3 production!”，然后再次运行 `pulumi up` 以检测更改并将修改后的文件上传到 S3：

```py
(venv) pulumi up
Previewing update (prod):

     Type                    Name        Plan       Info
     pulumi:pulumi:Stack     proj1-prod
 ~   └─ aws:s3:BucketObject  index.html  update     [diff: ~source]

Resources:
    ~ 1 to update
    11 unchanged

Do you want to perform this update? yes
Updating (prod):

     Type                    Name        Status      Info
     pulumi:pulumi:Stack     proj1-prod
 ~   └─ aws:s3:BucketObject  index.html  updated     [diff: ~source]

Outputs:
cloudfront_domain: "d3uhgbdw67nmlc.cloudfront.net"
log_bucket_id    : "cdn-log-bucket-53d8ea3"
web_bucket_id    : "s3-website-bucket-cde"
website_url      : "s3-website-bucket-cde.s3-website-us-east-1.amazonaws.com"

Resources:
    ~ 1 updated
    11 unchanged

Duration: 4s
```

使 CloudFront 分发的缓存失效以查看更改。

访问 [*https://www.devops4all.dev*](https://www.devops4all.dev)，查看消息：`Hello, S3 production!`

关于跟踪系统状态的 IaC 工具有一个注意事项：有时工具看到的状态与实际状态不同。在这种情况下，同步两种状态非常重要；否则，它们会越来越分离，你会陷入不敢再进行更改的境地，因为害怕会破坏生产环境。这就是为什么“Code”一词在“基础设施即代码”中占主导地位的原因。一旦决定使用 IaC 工具，最佳实践建议所有资源都通过代码进行配置，不再手动创建任何资源。保持这种纪律是很难的，但长远来看会带来回报。

# 练习

+   使用 [AWS Cloud Development Kit](https://aws.amazon.com/cdk) 配置相同的一组 AWS 资源。

+   使用 Terraform 或 Pulumi 从其他云服务提供商（如 Google Cloud Platform 或 Microsoft Azure）配置云资源。
