# CloudFormation Teaching Repo — Minecraft Server on EC2

A hands-on workshop repository for learning AWS CloudFormation.

You will explore a real-world template that provisions a Minecraft Java Edition
server on EC2, understand how each resource fits together, and see how
Infrastructure-as-Code is validated automatically using GitHub Actions.

> **This repo does not deploy anything.**
> It is a learning resource only. If you want to run the server yourself,
> see [Taking it further](#taking-it-further) at the bottom of this file.

---

## What is CloudFormation?

CloudFormation is AWS's Infrastructure-as-Code (IaC) service. Instead of
clicking through the AWS console to create resources, you describe everything
you want in a YAML file called a **template**. CloudFormation reads the
template and provisions all the resources for you in the correct order, as a
single unit called a **stack**.

**Why bother?**

- **Reproducible** — deploy the exact same setup in any AWS region or account
- **Version-controlled** — your infrastructure lives in Git alongside your code
- **Auditable** — every change is tracked
- **Reversible** — deleting the stack deletes all the resources it created

---

## Repo structure

```
.
├── minecraft-server.yaml          # The CloudFormation template (heavily annotated)
├── rules/
│   └── MinecraftServerRules.py    # Custom cfn-lint validation rules
└── .github/
    └── workflows/
        └── validate.yml           # GitHub Actions workflow — runs on every push/PR
```

---

## The template

Open `minecraft-server.yaml`. Every section and every resource has comments
explaining what it does and why it exists.

The template provisions:

| Resource | What it is |
|---|---|
| VPC | Your own isolated network in AWS |
| Subnet | A slice of that network in one availability zone |
| Internet Gateway | The "door" between your VPC and the public internet |
| Route Table | Tells the VPC where to send traffic |
| Security Group | Firewall rules — opens the Minecraft port and SSH |
| IAM Role + Instance Profile | Permissions for the EC2 instance to call AWS APIs |
| EC2 Instance | The virtual machine that runs the server |
| Elastic IP | A static public IP so the server address never changes |

**Key things to read in the template:**

1. The `Parameters` section — notice how values like instance size and server
   port are kept flexible rather than hard-coded
2. The `Conditions` section — see how `!If`, `!Not`, and `!Equals` work together
3. The `DependsOn` attributes — spot where CloudFormation needs explicit ordering
4. The `UserData` block — the bootstrap script that runs when the instance first boots
5. The `Outputs` section — how you retrieve useful info after the stack deploys

---

## Validation

Every push and pull request that touches the template or rules triggers the
GitHub Actions workflow in `.github/workflows/validate.yml`.

It runs **cfn-lint** — an open-source CloudFormation linter — which checks for:

- YAML syntax errors
- Invalid resource types or property names
- Wrong value types
- References to parameters or resources that don't exist
- Missing required properties

It also runs the **custom rules** in `rules/MinecraftServerRules.py`:

| Rule | Severity | What it checks |
|---|---|---|
| E9001 | Error | `ServerJarUrl` is not a placeholder value |
| E9002 | Error | `JavaMaxRam` is not too large for the chosen instance type |
| W9001 | Warning | `AllowSshCidr` is not wide open to the internet |
| W9002 | Warning | `EbsVolumeSize` is not too small |

No AWS account or credentials are needed to run any of this — cfn-lint runs
entirely on the GitHub Actions runner.

**Run validation locally:**

```bash
pip install cfn-lint
cfn-lint minecraft-server.yaml --append-rules rules/
```

---

## Taking it further

If you want to deploy this server yourself:

1. Copy `minecraft-server.yaml` into your own repo
2. Find the download URL for your chosen server JAR:
   - Vanilla: https://www.minecraft.net/en-us/download/server
   - Paper: https://papermc.io/downloads/paper
   - Fabric: https://fabricmc.net/use/server
3. Deploy via the AWS console:
   - CloudFormation → Create stack → Upload template
   - Fill in the parameters (ServerJarUrl is the important one)
4. Or deploy via the AWS CLI:
   ```bash
   aws cloudformation deploy \
     --stack-name minecraft-server \
     --template-file minecraft-server.yaml \
     --capabilities CAPABILITY_NAMED_IAM \
     --parameter-overrides \
       ServerJarUrl="https://your-jar-url-here" \
       AllowSshCidr="YOUR.IP.ADDRESS/32"
   ```
5. Once deployed, find your server IP in the CloudFormation Outputs tab
6. To tear everything down: CloudFormation → select the stack → Delete

> **Cost note:** The t4g.small instance costs roughly $0.017/hr (~$12/month
> running 24/7). The Elastic IP is free while associated with a running instance.
> Delete the stack when you are done to avoid ongoing charges.

---

## Further reading

- [AWS CloudFormation documentation](https://docs.aws.amazon.com/cloudformation/)
- [cfn-lint GitHub](https://github.com/aws-cloudformation/cfn-lint)
- [AWS for Games — Minecraft on EC2 blog post](https://aws.amazon.com/blogs/gametech/setting-up-a-minecraft-java-server-on-amazon-ec2/)
- [EC2 instance types](https://aws.amazon.com/ec2/instance-types/)
- [AWS Free Tier](https://aws.amazon.com/free/)
