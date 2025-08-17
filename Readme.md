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



aws cloudformation create-stack-set \
  --stack-set-name WebAppStackSet \
  --template-body file://vpc.yml \
  --capabilities CAPABILITY_NAMED_IAM \
  --administration-role-arn arn:aws:iam::272495906318:role/StackSetAdminRole \
  --execution-role-name StackSetExecutionRole \
  --profile Dev

  ## we assumed that we already created the Roles in dev and prod accounts

  ## then we create stackset instance

aws cloudformation create-stack-instances \
  --stack-set-name VpcStackSet \
  --accounts 272495906318 315089529175 \
  --regions us-east-1 \
  --operation-preferences FailureToleranceCount=0,MaxConcurrentCount=1



## Delete stack instance 
aws cloudformation delete-stack-instances \
  --stack-set-name VpcStackSet \
  --accounts 315089529175 \        ## you can add the second account if you want
  --regions us-east-1 \
  --retain-stacks \
  --no-retain-stacks \
  --operation-preferences FailureToleranceCount=0,MaxConcurrentCount=1 \
  --profile Dev   ## not mandatory if tou set the sso profile

## Delete the stack itself 
aws cloudformation delete-stack-set \
  --stack-set-name VpcStackSet


## creating stackset with nested stack
aws cloudformation create-stack-set \
  --stack-set-name WebAppStackSet \
  --template-url https://ironcore-stackset-templates.s3.us-east-1.amazonaws.com/main.yaml \
  --administration-role-arn arn:aws:iam::272495906318:role/StackSetAdminRole \
  --execution-role-name StackSetExecutionRole \
  --capabilities CAPABILITY_NAMED_IAM \
