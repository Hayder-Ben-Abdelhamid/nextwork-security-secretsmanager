# Secure Secrets with Secrets Manager

**Author:** Hayder Ben Abdelhamid  
**Email:** haider.abdelhamid@gmail.com  
**Role:** Cybersecurity Engineering Student  

This project is all about keeping sensitive AWS passwords and access keys safe. Instead of writing keys directly inside your code, you store them safely in AWS Secrets Manager and write code to fetch them automatically.  
You also learn how to use Git to clean up old project history so no hidden keys are left behind for hackers to find.

---

## 1. Introducing Today's Project!

In this project, I demonstrated how to use AWS Secrets Manager for secure credentials management. I'm doing this project to learn how to protect our credentials when connecting live (production) apps to our AWS environment, databases, etc.

### Key Tools and Concepts
* **AWS Services:** Secrets Manager, S3, IAM.
* **Other Tools:** GitHub, Git (Rebase).
* **Key Concepts:** GitHub secret scanning, the risks of hardcoding credentials, using `git rebase` to rewrite commit history, forking repositories, and secure configuration management.

### Project Reflection
This project took approximately 2 hours. The most challenging part was performing `git rebase` to clean up the commit history before pushing code to the repository. The most rewarding part was reviewing the git commit history and realizing how easily AWS credentials can leak through historical commits if not actively managed.

I chose to do this project today because I wanted to learn more about protecting and securing code using services like AWS Secrets Manager. This project met the goal of practicing best-practice credential management and helped build advanced Git skills along the way.

---

## 2. Hardcoding Credentials

In this project, a sample web app initially exposed a developer's AWS credentials (Access Key ID and Secret Access Key). Hardcoding credentials is unsafe because once code is made public in a repository, anyone can extract those credentials, access the AWS environment, delete resources, and steal data.

To practice safely, I used simulated test credentials to prevent exposing my actual live AWS keys during the project setup.

```python
# Unsafe hardcoded credentials example pattern
AWS_ACCESS_KEY_ID = "AKIAIOSFODNN7EXAMPLE"
AWS_SECRET_ACCESS_KEY = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
```

---

## 3. Using My Own AWS Credentials

As an extension of this project, I ran the web app locally using my own AWS credentials. To set up my virtual environment, I installed packages like `boto3`, `uvicorn`, and `fastapi`, which allow `app.py` to communicate with AWS.

When I first ran the app, I encountered an `InvalidAccessKeyId` error because test credentials do not point to a real, active AWS account.

```bash
# Error output encountered during initial local run
botocore.exceptions.ClientError: An error occurred (InvalidAccessKeyId) when calling the GetSecretValue operation: The AWS Access Key Id you provided does not exist in our records.
```

To resolve this, I updated `config.py` to use my active account credentials safely during initial local validation.

---

## 4. Pushing Insecure Code to GitHub

Once I updated the web app code, I forked the repository into a public GitHub account. A fork creates a new independent repository in your profile, whereas a clone copies code strictly to your local machine.

I linked my local repository to the remote fork and attempted to commit and push the changes:

```bash
git remote set-url origin https://github.com/Hayder-Ben-Abdelhamid/Secure-Secrets-Project.git
git add config.py
git commit -m "Add AWS credentials to config"
git push origin main
```

**GitHub Secret Scanning Block:**  
GitHub blocked my push because its automated secret scanning feature detected hardcoded AWS credentials. This is a vital security feature that prevents developers from accidentally leaking sensitive keys in public repositories.

---

## 5. Secrets Manager

AWS Secrets Manager is a service designed to securely store and retrieve secrets such as database credentials, API keys, and sensitive tokens. 

* **Secrets Rotation:** Automatically rotates secret values on a scheduled basis, minimizing the window of vulnerability if credentials are ever compromised.
* **Code Integration:** Secrets Manager provides ready-to-use sample code in Python, Java, JavaScript, and other languages to integrate secure retrieval functions directly into applications.

```python
# Example snippet provided by Secrets Manager for Python
import boto3
from botocore.exceptions import ClientError

def get_secret():
    secret_name = "TopSecretInfo"
    region_name = "us-east-1"

    session = boto3.session.Session()
    client = session.client(
        service_name='secretsmanager',
        region_name=region_name
    )
    try:
        get_secret_value_response = client.get_secret_value(
            SecretId=secret_name
        )
    except ClientError as e:
        raise e

    return get_secret_value_response['SecretString']
```

---

## 6. Updating the Web App Code

I updated `config.py` to retrieve secrets programmatically rather than storing them statically. The `get_secret()` function establishes a secure connection to Secrets Manager using `boto3` and pulls the secure payload into a runtime variable.

The configuration file now extracts the Access Key ID and Secret Access Key dynamically at runtime, ensuring no plaintext credentials exist in the source code.

---

## 7. Rebasing the Repository

Git rebasing modifies the commit history of a repository. I used an interactive rebase (`git rebase -i`) to drop the commit where hardcoded AWS credentials were originally introduced. This is crucial because historical commits can still expose secrets even if they are removed in the latest version of a file.

```bash
# Interactive rebase command to clean up historical commits
git rebase -i HEAD~3
```

**Resolving Conflicts:**  
A merge conflict occurred during rebasing because Git needed to reconcile changes made to `config.py` across commits. I manually edited `config.py` to ensure only the secure Secrets Manager implementation was retained.

**Final Verification:**  
After completing the rebase, I verified that the public GitHub repository did not expose any credentials. The code pushed successfully without triggering GitHub Secret Scanning blocks.
