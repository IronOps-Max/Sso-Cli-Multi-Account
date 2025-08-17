          ┌──────────────────────────────┐
          │         Dev Account          │
          │                              │
          │  ┌───────────────┐           │
          │  │ CodePipeline  │           │
          │  │---------------│           │
          │  │ Source        │ GitHub/CC │
          │  │ Build         │ CodeBuild │
          │  │ Deploy Dev    │ CodeDeploy│
          │  │ ManualApproval│           │
          │  │ Deploy Prod   │→ Assumes  │
          │  │               │ ProdRole │
          │  └───────────────┘           │
          │                              │
          │  ┌─────────────────────────┐ │
          │  │ StackSet / Nested Stacks │ │
          │  │-------------------------│ │
          │  │ VPC Stack               │ │
          │  │ - VPC, 2 Public + 2 Private Subnets │
          │  │ - IGW + NAT Gateways    │ │
          │  │ ALB/ASG Stack           │ │
          │  │ - ALB, Target Group     │ │
          │  │ - Launch Template & ASG│ │
          │  │ CodeDeploy Stack        │ │
          │  │ - App + Dev/Prod Groups│ │
          │  └─────────────────────────┘ │
          └──────────────────────────────┘
                     │
                     ▼
          ┌──────────────────────────────┐
          │         Prod Account         │
          │                              │
          │  ┌────────────────────────┐  │
          │  │ CodeDeploy Deployment  │  │
          │  │ Group (Prod ASG)       │  │
          │  │ Receives artifacts from│  │
          │  │ Dev pipeline           │  │
          │  └────────────────────────┘  │
          │                              │
          │  ┌────────────────────────┐  │
          │  │ ASG & EC2 Instances    │  │
          │  │ ALB + TG               │  │
          │  │ Same as Dev infra      │  │
          │  └────────────────────────┘  │
          └──────────────────────────────┘
