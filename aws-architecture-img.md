# AWS 構成図 (flowchart + 公式ロゴ画像埋め込み)

`flowchart` の image shape (`@{ img: ... }`) に Iconify の AWS ロゴ (SVG) を埋め込んだ版。
`architecture-beta` と違い、subgraph の入れ子・エッジラベル・スタイルが使えます。
画像は https://api.iconify.design/logos/<name>.svg から取得しています。

```mermaid
flowchart LR
    user@{ img: "https://api.iconify.design/logos/chrome.svg", label: "ユーザー", pos: "b", w: 56, h: 56, constraint: "on" }

    subgraph aws["AWS Cloud"]
        direction LR
        r53@{ img: "https://api.iconify.design/logos/aws-route53.svg", label: "Route 53", pos: "b", w: 56, h: 56, constraint: "on" }
        waf@{ img: "https://api.iconify.design/logos/aws-waf.svg", label: "AWS WAF", pos: "b", w: 56, h: 56, constraint: "on" }
        cf@{ img: "https://api.iconify.design/logos/aws-cloudfront.svg", label: "CloudFront", pos: "b", w: 56, h: 56, constraint: "on" }
        s3@{ img: "https://api.iconify.design/logos/aws-s3.svg", label: "S3 (静的)", pos: "b", w: 56, h: 56, constraint: "on" }

        subgraph vpc["VPC 10.0.0.0/16"]
            direction LR
            subgraph public["Public Subnet"]
                alb@{ img: "https://api.iconify.design/logos/aws-elb.svg", label: "ALB", pos: "b", w: 56, h: 56, constraint: "on" }
            end
            subgraph az1["Private Subnet (1a)"]
                ec2a@{ img: "https://api.iconify.design/logos/aws-ec2.svg", label: "EC2 #1", pos: "b", w: 56, h: 56, constraint: "on" }
            end
            subgraph az2["Private Subnet (1c)"]
                ec2c@{ img: "https://api.iconify.design/logos/aws-ec2.svg", label: "EC2 #2", pos: "b", w: 56, h: 56, constraint: "on" }
            end
            subgraph db["DB Subnet"]
                rds@{ img: "https://api.iconify.design/logos/aws-rds.svg", label: "RDS (Multi-AZ)", pos: "b", w: 56, h: 56, constraint: "on" }
            end
        end
    end

    user -- "DNS" --> r53
    r53 -. "ALIAS" .-> cf
    user -- "HTTPS 443" --> cf
    waf -. "ACL" .- cf
    cf -- "/static/*" --> s3
    cf -- "/* HTTPS" --> alb
    alb -- "HTTP 80" --> ec2a
    alb -- "HTTP 80" --> ec2c
    ec2a --> rds
    ec2c --> rds

    style aws fill:#F2F3F3,stroke:#232F3E,stroke-dasharray:5 5
    style vpc fill:#F6FBF7,stroke:#7AA116
    style public fill:#E6F4EA,stroke:#7AA116
    style az1 fill:#EAF2FB,stroke:#147EBA
    style az2 fill:#EAF2FB,stroke:#147EBA
    style db fill:#EEF0FA,stroke:#3B48CC
```

## PNG 出力

```sh
cd marmaidtest
npx -y @mermaid-js/mermaid-cli -p puppeteer-config.json \
  -i aws-architecture-img.md -o rendered/aws-architecture-img.png -b white -s 2
```

画像を外部 URL から取得するため、レンダリング時にネットワークが必要です。
GitHub 上でも image shape 自体は描画されますが、外部画像はプロキシ (camo) 経由になるため
表示されない場合があります。確実に共有したいときは PNG を使ってください。
