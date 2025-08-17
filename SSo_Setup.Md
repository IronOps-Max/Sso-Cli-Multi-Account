## Install the laster Vesion of AwsCLi

which aws
## If it points to /usr/bin/aws, then it was installed via apt.
sudo apt remove --purge awscli -y
sudo apt autoremove -y

# 🔹 2. If Installed via Official AWS Bundled Installer (zip)
sudo rm -rf /usr/local/aws-cli
sudo rm /usr/local/bin/aws

# 🔹 3. Verify Removal
aws --version


# Install New Version
sudo apt remove --purge awscli -y
sudo apt autoremove -y
sudo rm -rf /usr/local/aws-cli
sudo rm -f /usr/local/bin/aws
aws --version
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install


## 👉 If you had an older version installed via /usr/local/aws-cli, you might need to force reinstall:

sudo ./aws/install --update

# 🔹 5. Clean Up
rm -rf awscliv2.zip aws/

## aws configure sso

create dev and prod profiles each one poiting to the dev and prod account and also having full access to them

## 🔹  Test the Profile
aws sts get-caller-identity --profile dev|| prod

## 🔹  List All Profiles
aws configure list-profiles

## Show Active Profile

echo $AWS_PROFILE

## show which profile is Being Used
echo $AWS_PROFILE

## :🔹  Use a Profile for a Single Command

aws s3 ls --profile Dev

## 🔹Set a Profile for Your Current Shell Session

export AWS_PROFILE=Dev

Now every AWS CLI command in that shell will use dev-sso automatically:

aws s3 ls
aws ec2 describe-instances

To switch profile, just change the variable:

export AWS_PROFILE=Prod

## 🔹 Set a Permanent Default Profile (Optional)

If you always want one profile as the default, edit ~/.aws/config and add:

[default]
sso_start_url = https://mycompany.awsapps.com/start
sso_region = us-east-1
sso_account_id = 123456789012
sso_role_name = AdministratorAccess
region = us-east-1
output = json


Then running aws s3 ls (without --profile) will use this default.

## 🔹 Add a Bash/Zsh function (aws-switch)

You can create a helper in your ~/.bashrc or ~/.zshrc:

aws-switch() {
    if [ -z "$1" ]; then
        echo "Usage: aws-switch <profile-name>"
        echo "Available profiles:"
        aws configure list-profiles
    else
        export AWS_PROFILE=$1
        echo "Switched to AWS profile: $AWS_PROFILE"
    fi
}


Then reload your shell:

source ~/.bashrc   # or source ~/.zshrc

🔹 Usage
aws-switch dev-sso
aws sts get-caller-identity


Output shows you’re logged in under that account.

aws-switch prod-sso
aws s3 ls

## 🔹 Bonus: Show Current Profile

You can always check which profile you’re using:

echo $AWS_PROFILE


