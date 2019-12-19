# AWS CodeCommit

완전 관리형 형상관리 또는 버전관리 서비스로 고가용성, 확장성 및 안정성을 내재한 Git Repository를 제공합니다. 모든 소스코드는 암호화되어 전송 및 저장이 되고 Repository 크기는 무한대로 확장이 가능하며 어떤한 파일도 저장이 가능합니다. 다른 AWS 서비스들과 (AWS CodeBuild, CodeDeploy, CodePipeline, etc) 연계를 통해서 보다 나은 CI/CD 파이프라인을 구축할수 있습니다.

## 시작하기전에

1. 본 Hands-on lab은 AWS Seoul region 기준으로 작성되었습니다. Region을 Seoul (ap-northeast-2)로 변경 후 진행 부탁드립니다.
2. [AWS Credit 추가하기](https://aws.amazon.com/ko/premiumsupport/knowledge-center/add-aws-promotional-code/)
3. [Lab 환경 구축](https://ap-northeast-2.console.aws.amazon.com/cloudformation/home?region=ap-northeast-2#/stacks/quickcreate?templateURL=https://saltware-aws-lab.s3.ap-northeast-2.amazonaws.com/codecommit/lab.yml&stackName=codecommit)

## Repository에 연결

### HTTPS

1. CloudFormation으로 생성된 IAM user로 AWS Management Console 로그인

    ```text
    IAM user name: lead
    Password: asdf1234
    ```

2. AWS Management Console에서 좌측 상단에 있는 **[Services]** 를 선택하고 검색창에서 IAM를 검색하거나 **[Security, Identity, & Compliance]** 바로 밑에 있는 **[IAM]** 를 선택

3. IAM Dashboard에서  **[Users]** 클릭 &rightarrow; lead를 선택 &rightarrow; **[Security credentials]** &rightarrow; HTTPS Git credentials for AWS CodeCommit 밑에 있는 **[Generate credentials]** 를 클릭 후 User name과 Password를 메모

4. AWS Management Console 좌측 상단에 있는 **[Services]** 를 선택하고 검색창에서 ssm를 검색하고 **[Systems Manager]** 를 선택

5. Systems Manager Dashboard 왼쪽 패널 Instances & Nodes 섹션 아래에 있는 **[Session Manager]** 선택

6. **[Start Session]** &rightarrow; Instance Name: **lead** 선택 &rightarrow; **[Start Session]** 클릭

    - Home Directory로 이동

        ```bash
        cd
        ```

    - Git 설치

        ```bash
        sudo yum install git -y
        ```

    - Git 유저 설정

        ```bash
        git config --global user.email "you@example.com"
        git config --global user.name "Your Name"
        ```

    - Clone CodeCommit Repository

        ```bash
        git clone https://git-codecommit.ap-northeast-2.amazonaws.com/v1/repos/guess
        ```

    - 위에서 만든 Git credentials 입력

        ```bash
        Username for 'https://git-codecommit.ap-northeast-2.amazonaws.com':
        Password for 'https://lead-at-xxxxxxxxx@git-codecommit.ap-northeast-2.amazonaws.com':
        ```

    - Clone 해당 Github Repository

        ```bash
        git clone https://github.com/woowhoo/code-commit.git
        ```

    - 소스코드 복사

        ```bash
        cp code-commit/*.py guess/
        ```

    - Git credentials 저장을 원할경우 (Optional)

        ```bash
        git config credential.helper store
        ```

    - CodeCommit Repository로 소스코드를 push

        ```bash
        cd guess
        git add main.py test.py
        git commit -m "initial commit"
        git push
        ```

7. IAM Dashboard에서  **[Users]** 클릭 &rightarrow; lead를 선택 &rightarrow; **[Permissions]** 섹션 오른쪽에 있는 **[:heavy_plus_sign: Add inline policy]** 클릭후,
**Service** = CodeCommit, **Actions** = GitPush, **Resources** 탭에 있는 **[Add ARN]** 클릭 &rightarrow; **Region** = ap-northeast-2, **Repository name** = guess &rightarrow;  **[Add]**

8. **[Review Policy]** &rightarrow; **Name** = codecommit-lead-access &rightarrow; **[Create Policy]**

9. ```git push``` 재시도

### SSH

1. AWS Systems Manager Session Manager를 통해서 **dev** 인스턴스로 연결
    - RSA Key Pair 생성

        ```bash
        ssh-keygen
        ```

    - RSA Public Key를 클립보드로 복사

        ```bash
        cat ~/.ssh/id_rsa.pub
        ```

2. IAM Dashboard에서  **[Users]** 클릭 &rightarrow; dev를 선택 &rightarrow; **[Security credentials]** &rightarrow; HTTPS Git credentials for AWS CodeCommit 밑에 있는 **[Upload SSH public key]** 클릭 &rightarrow; 클립보드 내용 붙여넣기 &rightarrow; **[Upload SSH public key]** &rightarrow; SSH key ID를 메모

3. 다시 EC2 세션으로 돌아와서

    - Home Directory로 이동

        ```bash
        cd
        ```

    - Git 설치

        ```bash
        sudo yum install git -y
        ```

    - Git 유저 설정

        ```bash
        git config --global user.email "you@example.com"
        git config --global user.name "Your Name"
        ```

    - 로컬 SSH 연결 설정

        ```bash
        vi ~/.ssh/config
        ```

        ```text
        Host git-codecommit.*.amazonaws.com
        User Your-IAM-SSH-Key-ID-Here
        IdentityFile ~/.ssh/id_rsa
        ```

        ```bash
        chmod 600 ~/.ssh/config
        ```

    - Clone CodeCommit Repository

        ```bash
        git clone ssh://git-codecommit.ap-northeast-2.amazonaws.com/v1/repos/guess
        ```

4. 어플리케이션을 1 ~ 20까지의 숫자를 맞추는걸로 수정하고 Git Repository에 변경사항 반영

## Branching

1. 다시 EC2 세션으로 돌아와서 Git Branch 생성 후 Origin으로 Push

    ```bash
    git checkout -b feature-max20
    git push -u origin feature-max20
    ```

2. IAM Dashboard에서  **[Users]** 클릭 &rightarrow; dev를 선택 &rightarrow; **[Permissions]** 섹션 아래 **[Add permissions]** &rightarrow; **[Attach existing policies directly]** &rightarrow; :white_check_mark: AWSCodeCommitPowerUser 선택 &rightarrow; **[Next: Review]** &rightarrow; **[Add permissions]**

3. **[:heavy_plus_sign: Add inline policy]** 클릭 &rightarrow; **JSON** 선택 후 아래 내용 붙여넣고 **[Review policy]** 

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Deny",
                "Action": [
                    "codecommit:GitPush",
                    "codecommit:DeleteBranch",
                    "codecommit:PutFile",
                    "codecommit:MergePullRequestByFastForward"
                ],
                "Resource": "arn:aws:codecommit:ap-northeast-2:<ACCOUNT_NUMBER>:guess",
                "Condition": {
                    "StringEqualsIfExists": {
                        "codecommit:References": [
                            "refs/heads/master"
                        ]
                    },
                    "Null": {
                        "codecommit:References": false
                    }
                }
            }
        ]
    }
    ```

4. **Name** = DenyChangesToMaster &rightarrow; **[Create policy]**

5. 다시 EC2 세션으로 돌아와서 Git Branch를 Origin으로 Push 재시도

    ```bash
    git push -u origin feature-max20
    ```

## Pull Request & Merging branch

1. `dev` IAM user로 AWS Management Console 로그인

    ```text
    IAM user name: dev
    Password: asdf1234
    ```

2. AWS Management Console에서 좌측 상단에 있는 **[Services]** 를 선택하고 검색창에서 CodeCommit을 검색하거나 **[Developer Tools]** 밑에 있는 **[CodeCommit]** 를 선택

3. `guess` Repository를 선택 &rightarrow; 왼쪽 패널에서 **[Branches]** &rightarrow; **[Create pull request]** 클릭 &rightarrow; **Destination** = master, **Source** = feature-max20 &rightarrow; **[Compare]**

4. **Title** = NEW-01 최댓값 변경, **Description** = 사용자 XXX의 요청으로 최댓값을 20으로 변경 &rightarrow; **[Create pull request]**

5. `lead` IAM user로 AWS Management Console 로그인하고 AWSCodeCommitPowerUser 권한 부여

6. CodeCommit Dashboard에서 Pull requests 섹션으로 이동

7. **NEW-01 최댓값 변경** Pull request를 선택 &rightarrow; **[Approve]** &rightarrow; **[Merge]** &rightarrow; **[Merge pull request]**

8. Master branch의 소스코드에 변경사항이 적용 됬는지 확인

