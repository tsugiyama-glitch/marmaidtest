# AWS 構成図 (CloudFront → ALB → EC2)

Mermaid の `flowchart` を使った AWS 構成図のサンプルです。
GitHub / VS Code (Markdown Preview Mermaid Support) などでそのまま描画できます。

## 構成図

```mermaid
flowchart LR
    user(["👤 ユーザー<br/>(ブラウザ)"])

    subgraph aws["☁️ AWS Cloud"]
        direction LR

        r53["Route 53<br/>example.com"]
        cf["CloudFront<br/>Distribution"]
        waf["AWS WAF"]
        s3[("S3<br/>静的コンテンツ")]

        subgraph region["Region: ap-northeast-1"]
            direction LR
            subgraph vpc["VPC 10.0.0.0/16"]
                direction LR

                subgraph public["Public Subnets"]
                    alb["Application<br/>Load Balancer"]
                end

                subgraph az1["AZ: ap-northeast-1a<br/>Private Subnet"]
                    ec2a["EC2<br/>Web/App #1"]
                end

                subgraph az2["AZ: ap-northeast-1c<br/>Private Subnet"]
                    ec2c["EC2<br/>Web/App #2"]
                end

                subgraph db["DB Subnets"]
                    rds[("RDS<br/>Multi-AZ")]
                end
            end
        end
    end

    user -- "DNS 解決" --> r53
    r53 -. "CNAME / ALIAS" .-> cf
    user -- "HTTPS (443)" --> cf
    waf -. "アクセス制御" .- cf
    cf -- "/static/* (Origin: S3)" --> s3
    cf -- "/* (Origin: ALB)<br/>HTTPS" --> alb
    alb -- "HTTP (80)<br/>Target Group" --> ec2a
    alb -- "HTTP (80)<br/>Target Group" --> ec2c
    ec2a --> rds
    ec2c --> rds

    %% スタイル定義 (AWS っぽい配色)
    classDef edge fill:#8C4FFF,stroke:#5A2D9F,color:#fff
    classDef compute fill:#ED7100,stroke:#B85600,color:#fff
    classDef network fill:#8C4FFF,stroke:#5A2D9F,color:#fff
    classDef storage fill:#3F8624,stroke:#2C5E19,color:#fff
    classDef database fill:#3B48CC,stroke:#27308A,color:#fff
    classDef security fill:#DD344C,stroke:#A02335,color:#fff
    classDef actor fill:#fff,stroke:#232F3E,color:#232F3E

    class cf,r53 edge
    class alb network
    class ec2a,ec2c compute
    class s3 storage
    class rds database
    class waf security
    class user actor

    style aws fill:#F2F3F3,stroke:#232F3E,stroke-dasharray:5 5
    style region fill:#FFFFFF,stroke:#00A4A6,stroke-dasharray:5 5
    style vpc fill:#F6FBF7,stroke:#7AA116
    style public fill:#E6F4EA,stroke:#7AA116
    style az1 fill:#EAF2FB,stroke:#147EBA
    style az2 fill:#EAF2FB,stroke:#147EBA
    style db fill:#EEF0FA,stroke:#3B48CC
```

## シンプル版 (最小構成)

```mermaid
flowchart LR
    user(["ユーザー"]) -- HTTPS --> cf["CloudFront"]
    cf -- HTTPS --> alb["ALB"]
    alb -- HTTP --> ec2["EC2"]

    classDef aws fill:#ED7100,stroke:#B85600,color:#fff
    class cf,alb,ec2 aws
```

## リクエストフロー (シーケンス図)

```mermaid
sequenceDiagram
    autonumber
    actor U as ユーザー
    participant CF as CloudFront
    participant ALB as ALB
    participant EC2 as EC2

    U->>CF: GET / (HTTPS)
    alt キャッシュヒット
        CF-->>U: 200 OK (cache)
    else キャッシュミス
        CF->>ALB: GET / (HTTPS, Origin)
        ALB->>EC2: GET / (HTTP, Target Group)
        EC2-->>ALB: 200 OK
        ALB-->>CF: 200 OK
        CF-->>U: 200 OK (キャッシュ保存)
    end
```

## 補足

- `subgraph` で AWS Cloud / Region / VPC / Subnet の入れ子を表現しています。
- `classDef` / `style` で AWS のサービスカテゴリ配色 (Compute: オレンジ、Network: 紫、Storage: 緑、DB: 青) を模しています。
- 公式の AWS アイコンを使いたい場合は Mermaid 単体では困難なため、`architecture-beta` 図 (Mermaid v11+) や PlantUML / diagrams (Python) の利用を検討してください。
