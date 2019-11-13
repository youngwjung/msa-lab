# MSA 맛보기
## Architecture Overview
![dsfdsfsdfsdf](https://saltware-aws-lab.s3.ap-northeast-2.amazonaws.com/msa/img/msa-101-architecture.png)

## 시작하기전에
1. 본 Hands-on lab은 AWS Seoul region 기준으로 작성되었습니다. Region을 Seoul (ap-northeast-2)로 변경 후 진행 부탁드립니다.
2. [AWS Credit 추가하기](https://aws.amazon.com/ko/premiumsupport/knowledge-center/add-aws-promotional-code/)
3. [Lab 환경 구축](https://ap-northeast-2.console.aws.amazon.com/cloudformation/home?region=ap-northeast-2#/stacks/quickcreate?templateURL=https://saltware-aws-lab.s3.ap-northeast-2.amazonaws.com/msa/msa-hol.yml&stackName=msa-lab)

## Application 실행
1. AWS Management Console 좌측 상단에 있는 **[Services]** 를 선택하고 검색창에서 ssm를 검색하고 **[Systems Manager]** 를 선택
2. Systems Manager Dashboard 왼쪽 패널 Instances & Nodes 섹션 아래에 있는 **[Session Manager]** 선택
3. **[Start Session]** &rightarrow; Instance Name: **react-app** 선택 &rightarrow; **[Start Session]** 클릭
  - Root 환경으로 전환
    ```bash
    sudo -i
    ```

  - Node.js App 설정 파일 열기
    ```bash
    vi /home/ec2-user/react-ecommerce-node/config.js
    ```

  - **DB_URL** = <RDS_ENDPOINT> 로 변경 (RDS Console에서 확인 가능)
    ```javascript
    module.exports = {
      DB_URL: 'react-mysql.xxxxxxxx.ap-northeast-2.rds.amazonaws.com'
    };
    ```

  - Node.js 서버 시작
    ```bash
    forever start /home/ec2-user/react-ecommerce-node/app.js
    ```

  - React App 설정 파일 열기
    ```bash
    vi /home/ec2-user/react-ecommerce/src/config/config.js
    ```

  - **API_SERVER** = http://<EC2_PUBLIC_IP>:5000 로 변경 (EC2 Console에서 확인 가능)
    ```javascript
    const API_SERVER = 'http://123.123.123.123:5000'
    ```

  - React 빌드 생성
    ```bash
    cd /home/ec2-user/react-ecommerce && yarn build
    ```

  - React 서버 시작
    ```bash
    cd /home/ec2-user/react-ecommerce && /usr/local/bin/serve -s build --listen 80
    ```

## Serverless backend
![](https://saltware-aws-lab.s3.ap-northeast-2.amazonaws.com/msa/img/serverless_diagram.png)

### Lambda
- 이벤트에 대한 응답으로 코드를 실행하고 자동으로 기본 컴퓨팅 리소스를 관리하는 서버리스 컴퓨팅
- Java, Python, Node.js, C#, Go, Ruby
- 완전 관리형 서비스 (서버와 운영체제 유지 관리, Capacity Provisioning & Autoscaling, 로깅 및 모니터링, 보안패치 및 코드배포 등등)
- Event-driven compute: CloudWatch, S3, DynamoDB, SNS, etc
- HTTP request handler: API Gateway, Application Load Balancer

### API Gateway
- API를 생성, 게시, 유지관리 및 모니터링 할수있는 완전관리형 서비스
- REST 및 WebSocket API 지원
- Integrated with IAM, Cognito, CloudFront, etc

### DynamoDB
- NoSQL Database
- Single-digit millisecond performance at any scale
- Support in-memory cache, global replication, point-in-time recovery

### REST API 구성 (/event)

1. AWS Management Console에서 좌측 상단에 있는 **[Services]** 를 선택하고 검색창에서 DynamoDB를 검색하거나 **[Database]** 밑에 있는 **[DynamoDB]** 를 선택

2. DynamoDB Dashboard에서  **[Create table]** 클릭후,
  **Table name** = event,
  **Primary key** = email,
  **Use default settings** = :white_check_mark:,
  **[Create]** 클릭

3. AWS Management Console에서 좌측 상단에 있는 **[Services]** 를 선택하고 검색창에서 SES를 검색하거나 **[Customer Engagement]** 밑에 있는 **[Simple Email Service]** 를 선택 &rightarrow; Asia Pacific (Sydney) 선택

4. SES Dashboard 왼쪽 패널에서 **Email Addresses** 를 선택 &rightarrow; **[Verify a New Email Address]** &rightarrow; 본인의 이메일 주소 입력 후 **[Verify This Email Address]** 클릭

5. 메일박스에서 **Amazon Web Services – Email Address Verification Request in region Asia Pacific (Sydney)** 이메일 확인후 인증 링크 클릭

6. SES Console에서 인증받은 이메일의 Verification Status가 **verified**로 변경되었는지 확인

7. DynamoDB Dashboard에서  **[Create table]** 클릭후,
  **Table name** = event,
  **Primary key** = email,
  **Use default settings** = :white_check_mark:,
  **[Create]** 클릭

8. AWS Management Console에서 좌측 상단에 있는 **[Services]** 를 선택하고 검색창에서 Lambda를 검색하거나 **[Compute]** 밑에 있는 **[Lambda를]** 를 선택

9. Lambda Dashboard에서  **[Create function]** 클릭후,
  **Function name** = event_handler,
  **Runtime** = Python 3.7,
  **[Create function]** 클릭

10. 좌측 하단에 있는 **[Execution role]** 에서 **View the event_handler-role-xxxx** on the IAM console 를 선택

11. **[Add inline policy]** 선택 후,
    **Service** = DynamoDB,
    **Actions** = PutItem,
    **Resources** = :white_check_mark: Specific &rightarrow; **[Add ARN]**,
    **Region** = ap-northeast-2, **Table name** = event &rightarrow; **[Add]**,
    **[Add additional permissions]** 클릭,
    **Service** = X-ray,
    **Actions** = :white_check_mark: Write,
    **Resources** = :white_check_mark: All resources;
    **[Add additional permissions]** 클릭,
    **Service** = SES,
    **Actions** = SendEmail,
    **Resources** = :white_check_mark: All resources;
    **[Review policy]** 클릭,
    **Name** = event_handler,
    **[Create policy]** 클릭

12. Lambda Console로 돌아와서 우측 하단에 있는 **[AWS X-Ray]** 에서 :white_check_mark: **Active tracing** Specific &rightarrow; **[Save]** 클릭

13. 아래 코드블록을 Lambda에 복사 후, **[Save]** 클릭
  ```python
  import json
  import boto3

  def lambda_handler(event, context):

      dynamodb = boto3.resource('dynamodb')
      table = dynamodb.Table('event')

      response = table.put_item(
          Item=event    
      )
      if (response['ResponseMetadata']['HTTPStatusCode'] == 200):
          print ("successful")
          client = boto3.client('ses', region_name='ap-southeast-2')

          sesResponse = client.send_email(
              Destination={
                  'ToAddresses': [event['email']],
              },
              Message={
                  'Body': {
                      'Text': {
                          'Charset': 'UTF-8',
                          'Data': 'Registered',
                      },
                  },
                  'Subject': {
                      'Charset': 'UTF-8',
                      'Data': 'Thank you',
                  },
              },
              Source=event['email'],
          )

          print (sesResponse)
          print ("email sent")

      return {
          'statusCode': 200,
          'body': json.dumps(response)
      }
  ```

14. Lambda Console로 돌아가서 **[Designer]** 섹션에 있는 **+ Add trigger** 클릭 &rightarrow; Dropdown 리스트에서 **API Gateway** 선택 &rightarrow; **API** = Create a new API, **Security** = Open &rightarrow; **[Add]** 클릭

15. Lambda Console의  **[API Gateway]** 섹션에서 **event_handler-API** 클릭

16. **/event_handler** 선택 &rightarrow; Actions &rightarrow; Create Method &rightarrow; POST 선택 &rightarrow; :white_check_mark: &rightarrow; **Integration type** = Lambda Function, **Lambda Region** = ap-northeast-2, **Lambda Function** = event_handler &rightarrow; **[Save]** 클릭

17. **/event_handler** 밑에 **POST** 선택 &rightarrow; Actions &rightarrow; Enable CORS &rightarrow; **[Enable CORS and replace existing CORS headers]** 클릭 &rightarrow; **[Yes, replace existing values]** 클릭

18. **/event_handler** 선택 &rightarrow; Actions &rightarrow; Deploy API &rightarrow; **Deployment stage** = default &rightarrow; **[Deploy]** 클릭

19. Stages &rightarrow; Default &rightarrow; /event_handler &rightarrow; POST &rightarrow; **[Invoke_URL]**를 클립보드에 복사

20. Session Manager를 통해서 EC2 인스턴스에 접속후,
  - React App 설정 파일 열기
    ```bash
    vi /home/ec2-user/react-ecommerce/src/config/config.js
    ```

  - **EVENT_REGISTER**: <API_GATEWAY_INVOKE_URL>로 변경
    ```javascript
    EVENT_REGISTER: `https://xxxxx.execute-api.ap-northeast-2.amazonaws.com/default/event_handler`
    ```

  - React 빌드 생성
    ```bash
    cd /home/ec2-user/react-ecommerce && yarn build
    ```

  - React 서버 시작
    ```bash
    cd /home/ec2-user/react-ecommerce && /usr/local/bin/serve -s build --listen 80
    ```

21. 웹사이트에 접속후 Event 페이지로 이동 후 폼 작성 및 제출 &rightarrow; DynamoDB 테이블에 Record 추가 됐는지 확인 & 이벤트 접수 메일 왔는지 확인

## Scalable Frontend
### Static Website Hosting on S3
  - HTML, CSS, Javascript 및 기타 정적(Image, text) 파일로 구성된 정적 사이트 호스팅
  - Route53와 연계해서 Custom Domain으로 연결 가능
  - Highly available & scalable Serverless Frontend 구성

1. AWS Management Console에서 좌측 상단에 있는 **[Services]** 를 선택하고 검색창에서 S3를 검색하거나 **[Storage]** 바로 밑에 있는 **[S3]** 를 선택

2. S3 Dashboard에서  **[Create bucket]** 클릭후, **Bucket name** = [임의의 문자 및 숫자].saltware.io, **Region** = Asia Pacific (Seoul), **[Create]** 클릭

3. 해당 Bucket 클릭 &rightarrow; **[Properties]** &rightarrow; **[Static website hosting]** &rightarrow; :white_check_mark: Use this bucket to host a website, **Index document** = index.html, **Error document** = index.html &rightarrow; **[Save]**

4. 해당 Bucket 클릭 &rightarrow; **[Permissions]** &rightarrow; **[Block public access]** &rightarrow; **[Edit]** &rightarrow;  uncheck **Block _all_ public access** &rightarrow; **[Save]**

5. **[Permissions]** &rightarrow; **[Bucket Policy]** &rightarrow; 아래 Policy 블록을 Bucket policy editor에 붙여놓고 **[Save]**
  ```json
  {
    "Version":"2012-10-17",
    "Statement":[
      {
        "Sid":"AddPerm",
        "Effect":"Allow",
        "Principal": "*",
        "Action":["s3:GetObject"],
        "Resource":["arn:aws:s3:::<BUCKET_NAME>/*"]
      }
    ]
  }
  ```

### Continuous Deployment
  - React 소스코드를 Github에 호스팅
  - AWS CodePipeline을 이용해서 코드 변경시 배포 자동화 구성
  - AWS Cloud9을 이용해서 개발환경 구성

1. 해당 [Git Repository](https://github.com/woowhoo/react-ecommerce)를 Fork (GitHub 계정 필수)

2. AWS Management Console에서 좌측 상단에 있는 **[Services]** 를 선택하고 검색창에서 CodePipeline를 검색하거나 **[Developer Tools]** 밑에 있는 **[CodePipeline]** 를 선택

3. **[Create pipeline]** &rightarrow; **Pipeline name** = msa-frontend, **Service role** = New service role &rightarrow; **[Next]** &rightarrow; **Source provider** = GitHub &rightarrow; **[Connect to GitHub]** &rightarrow; **Repository** = Step 1에서 Forking한 Repository, **Branch** = master &rightarrow; **[Next]** &rightarrow; **Build provider** = AWS CodeBuild &rightarrow; **[Create project]**

4. **Project name** = msa-frontend, **Environment image** = Managed Image, **Operating system** = Ubuntu, **Runtime(s)** = Standard, **Image** = aws/codebuild/standard:2.0, **Service role** = New service role, **Build specifications** = Insert build commands &rightarrow; **[Switch to editor]** &rightarrow; 아래 커맨드블록을 Build commands에 붙여놓고 **[Continue to CodePipeline]**
  ```yaml
  version: 0.2

  phases:
      install:
          runtime-versions:
              nodejs: 10
          commands:
              # Yarn installation
              - curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
              - echo "deb http://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
              - apt-get -y update
              - apt-get install -y yarn
              # Installing react and serverless dependencies
              - yarn install

      build:
          commands:
              - yarn run build

  artifacts:
      files:
          - '**/*'
      base-directory: 'build'
  ``` 

5. **[Next]** &rightarrow; **Deploy provider** = AWS S3, **Bucket** = 바로 전에 생성한 S3 Bucket, :white_check_mark: Extract file before deploy &rightarrow; **[Next]** &rightarrow; **[Create pipeline]**

6. AWS Management Console에서 좌측 상단에 있는 **[Services]** 를 선택하고 검색창에서 Cloud9를 검색하거나 **[Developer Tools]** 밑에 있는 **[Cloud9]**를 선택 &rightarrow; Asia Pacific (Tokyo) 선택

7. **[Create environment]** &rightarrow; **Name** = dev &rightarrow; **[Next step]** &rightarrow; **Environment type** = :white_check_mark: Create a new instance for environment (EC2), **Instance type** = t2.micro (1 GiB RAM + 1 vCPU), **Platform** = Amazon Linux &rightarrow; **[Next step]** &rightarrow; **[Create environment]**

8. Forking한 Repository를 Cloud9으로 환경으로 Clone
  ```bash
  git clone http://<REPOSITORY_URL>
  ```

9. React App 설정파일 수정 (API_SERVER, EVENT_REGISTER)

10. 수정한 코드를 Commit & Push
  ```bash
  git add .
  git commit -m "add API endpoints to config"
  git push
  ```

## AppSync
![](https://saltware-aws-lab.s3.ap-northeast-2.amazonaws.com/msa/img/appsync.png)

1. AWS Management Console에서 좌측 상단에 있는 **[Services]** 를 선택하고 검색창에서 AppSync를 검색하거나 **[Mobile]** 밑에 있는 **[AWS AppSync]**를 선택

2. **[Create API]** &rightarrow; **Import DynamoDB Table** &rightarrow; **[Start]** &rightarrow; **Table name** = event, **Create or use an existing role** = :white_check_mark: New role &rightarrow; **[Import]**

3. **Model Name** = Event &rightarrow; **[Add field]**, **Name** = name, **Type** = String, **Required** = :white_check_mark: &rightarrow; **[Add field]**, **Name** = mobile, **Type** = String, **Required** = :white_check_mark: &rightarrow; **[Create]** &rightarrow; **API Name** = Event &rightarrow; **[Create]**

4. src/index.js 파일을 열고 AppSync Endpoint 정보를 입력, 관련정보는 **[Settings]**에서 확인
  ```javascript
  const client = new AWSAppSyncClient({
    url: 'https://xxxxx.appsync-api.ap-northeast-2.amazonaws.com/graphql',
    region: 'ap-northeast-2',
    auth: {
      type: 'API_KEY',
      apiKey: 'xxxxxxxxxxxxxxxxxx',
    }
  })
  ```

5. src/components/event/event.component.jsx 파일을 열고 아래와 같이 35줄을 주석처리하고 36줄의 주석처리를 해제
  ```javascript
  // await events.register(name, email, mobile);
  this.props.onAdd(input);
  ```

6. 수정한 코드를 Commit & Push

### Event-driven microservice

1. AWS Management Console에서 좌측 상단에 있는 **[Services]** 를 선택하고 검색창에서 Lambda를 검색하거나 **[Compute]** 밑에 있는 **[Lambda]** 를 선택

2. Lambda Dashboard에서  **[Create function]** 클릭후,
  **Function name** = send_email,
  **Runtime** = Python 3.7,
  **[Create function]** 클릭

3. 좌측 하단에 있는 **[Execution role]** 에서 **View the send_email-role-xxxx** on the IAM console 를 선택

4. **[Attach policies]** &rightarrow; DynamoDB 검색 &rightarrow; :white_check_mark: **AWSLambdaInvocation-DynamoDB** 선택 &rightarrow; **[Attach policy]**

5. Inline policy를 이용해서 해당 IAM Role에 SES SendEmail 권한 부

6. 아래 코드블록을 Lambda에 복사 후, **[Save]** 클릭
  ```python
  import json
  import boto3

  def lambda_handler(event, context):
      #print("Received event: " + json.dumps(event, indent=2))
      for record in event['Records']:
          if (record['eventName'] == 'INSERT'):
              email = record['dynamodb']['Keys']['email']['S']

              client = boto3.client('ses', region_name='ap-southeast-2')
              sesResponse = client.send_email(
                  Destination={
                      'ToAddresses': [email],
                  },
                  Message={
                      'Body': {
                          'Text': {
                              'Charset': 'UTF-8',
                              'Data': 'Registered',
                          },
                      },
                      'Subject': {
                          'Charset': 'UTF-8',
                          'Data': 'Thank you',
                      },
                  },
                  Source=email,
              )
              print (sesResponse)
              print ("email sent")
      return 'OK'
  ```

7. AWS Management Console에서 좌측 상단에 있는 **[Services]** 를 선택하고 검색창에서 DynamoDB를 검색하거나 **[Database]** 밑에 있는 **[DynamoDB]** 를 선택

8. DynamoDB Console에서 **[Tables]** &rightarrow; event 선택

9. Overview 탭 아래에 있는 **Stream details** 에서 **[Manage Stream]** 클릭 &rightarrow; :white_check_mark: New image &rightarrow; **[Enable]**

10. Trigger 탭 선택 &rightarrow; **[Create trigger]** &rightarrow; **[Existing Lambda function]** &rightarrow; **Function** = send_mail, **Batch size** = 1, **Enable trigger** = :white_check_mark: &rightarrow; **[Create]**

## AWS Cognito

1. AWS Management Console에서 좌측 상단에 있는 **[Services]** 를 선택하고 검색창에서 Cognito를 검색하거나 **[Security, Identity, & Compliance]** 밑에 있는 **[Cognito]** 를 선택

2. **[Manage User Pools]** &rightarrow; **[Create a user pool]** &rightarrow; **Pool name** = msa-app &rightarrow; **[Review defaults]**

3. **[Choose username attributes..]** &rightarrow; :white_check_mark: Email address or phone number &rightarrow; :white_check_mark: Allow email addresses &rightarrow; 왼쪽 패널에서 **Review** 선택 &rightarrow; **[Create pool]**

4. 왼쪽 패널에서 **App clients** 선택 &rightarrow; **[Add an app client]** &rightarrow; **App client name** = msa-app, **Generate client secret** = 체크해제 &rightarrow; **[Create app client]**

5. 좌측 상단에 있는 **Federated Identities** 선택 &rightarrow; **Identity pool name** = msa app &rightarrow; Authentication providers 탭 펼치기 &rightarrow; Cognito 탭 아래 **User Pool ID** , **App client id** 입력 &rightarrow; **[Create Pool]** &rightarrow; **[Allow]**

6. Cloud9에서 src/config/cognito.js 파일을 열고 Cognito Endpoint 정보를 입력,
  ```javascript
  export default {
    cognito: {
      REGION: "ap-northeast-2",
      USER_POOL_ID: "ap-northeast-2_xxxxx",
      APP_CLIENT_ID: "xxxxxxx",
      IDENTITY_POOL_ID: "ap-northeast-2:xxxx-xxxx-xxxxxxx"
    }
  };
  ```

7. Cloud9에서 src/components/sign-up/sign-up.component.jsx 파일을 열고 아래와 같이 38-54줄을 주석처리를 해제하고, 58줄 주석처리, 60줄 주석처리 해제
  ```javascript
  if(showVerification == false) {

    const currentUser = await Auth.signUp({
      username: email,
      password: password
    });

    this.setState({
      displayName: displayName,
      email: email,
      password: password,
      confirmPassword: confirmPassword,
      verificationCode: verificationCode,
      showVerification: true
    });
    return;
  }

  try {

    // const currentUser = await user.register(email, password);
    // console.log(verificationCode);
    const currentUser = await Auth.confirmSignUp(email, verificationCode)
  ```

8. 수정한 코드를 Commit & Push

9. 웹사이트에 접속 후, 신규 계정 생성 및 로그인 테스트 (Verification Code는 가입 이메일로 발송)

10. Cognito User Pool에 신규 유저가 생성됬는지 확인

## Container
![](https://saltware-aws-lab.s3.ap-northeast-2.amazonaws.com/msa/img/container.png)

### Docker Image 생성

1. AWS Management Console에서 좌측 상단에 있는 **[Services]** 를 선택하고 검색창에서 ECR를 검색하거나 **[Compute]** 밑에 있는 **[ECR]** 를 선택

2. **[Get Started]** &rightarrow; **Repository name** = msa-node &rightarrow; **[Create repository]**

3. Cloud9 에서 Node.js backend repository 복제
  ```bash
  cd ~/environment && git clone https://github.com/woowhoo/react-ecommerce-node.git
  ```

4. Cloud9에서 react-ecommerce-node/config.js 파일을 열고 **DB_URL** = <RDS_ENDPOINT> 로 변경 (RDS Console에서 확인 가능)
  ```javascript
  module.exports = {
    DB_URL: 'react-mysql.xxxxxxxx.ap-northeast-2.rds.amazonaws.com'
  };
  ```

5. 복제한 Repository 루트 디렉토리에 Dockerfile 파일 생성후 아래 내용 복사
  ```docker
  FROM node:10

  # Create app directory
  WORKDIR /usr/src/app

  # Bundle app source
  COPY . .

  EXPOSE 5000
  CMD [ "node", "app.js" ]
  ```

6. ECR Dashboard 에서 위에서 만든 Repository 선택 &rightarrow; 우측 상단에 있는 **[View push commands]** 클릭

7. Cloud9에서 복제한 Repository 루트 디렉토리로 이동후 위의 커맨드를 순서대로 실행

8. ECR Dashboard를 refresh하고 Docker 이미지가 추가 됬는지 확인

### Fargate 구성
![](https://saltware-aws-lab.s3.ap-northeast-2.amazonaws.com/msa/img/fargate.png)

1. AWS Management Console에서 좌측 상단에 있는 **[Services]** 를 선택하고 검색창에서 ECS를 검색하거나 **[Compute]** 밑에 있는 **[ECS]** 를 선택

2. 좌측 패널에서 **Task Definitions** 클릭 &rightarrow; **[Create new Task Definition]** &rightarrow; FARGATE 선택 후 **[Next step]** &rightarrow; **Take Definition Name** = msa-node, **Task memory (GB)** = 1GB, **Task CPU (vCPU)** = 0.5 vCPU

3. **[Add container]** &rightarrow; **Container name** = msa-node, **Image** = <Image_URI> ECR에서 확인, **Port mappings** = 5000, &rightarrow; **[Add]** &rightarrow; **[Create]** &rightarrow; **[View task definition]**

4. 좌측 패널에서 **Clusters** 클릭 &rightarrow; **[Create Cluster]** &rightarrow; Networking only 선택 후 **[Next step]** &rightarrow; **Cluster Name** = msa-node &rightarrow; **[Create]** &rightarrow; **[View Cluster]**

5. **Services** 탭 아래 **[Create]** 클릭 &rightarrow;
  **Launch type** = :white_check_mark: FARGATE,
  **Task Definition** = Family: msa-node | Revision: 1(latest),
  **Service name** = msa-node,
  **Number of tasks** = 1,
  **[Next step]**

6. **Cluster VPC** = msa-vpc(10.1.0.0/16),
  **Subnets** = 전부 선택,
  **Security groups** = **[Edit]** &rightarrow; :white_check_mark: Select existing security group, :white_check_mark: ec2-sg &rightarrow; **[Save]**

7. 나머지 옵션들은 다 기본값으로 두고 **[Next step]**

8. **Service Auto Scaling** = Do not adjust the service’s desired count &rightarrow; **[Next step]** &rightarrow; **[Create Service]** &rightarrow; **[View Service]**

9. Last status가 **RUNNING** 인지 확인

10. Task ID 클릭 &rightarrow; **Public IP**를 클립보드에 복사

11. Cloud9 에서 React App 설정파일 수정 (API_SERVER)
  ```javascript
  const API_SERVER = 'http://<Public_IP>:5000'
  ```

12. 수정한 코드를 Commit & Push

13. EC2 인스턴스를 정지하고 앱이 정상적으로 작동하는지 확인

## Clean up
1. ECS - Cluster, Task Definition, ECR Repository
2. Cognito - Domain, User Pool, Identity Pool
3. Lambda - event_handler, send_mail
4. AppSync
5. DynamoDB
6. Cloud9 - Tokyo Region
7. CodePipeline
8. CodeBuild
9. GitHub Repository
10. S3 Bucket
11. API Gateway
12. SES - Email Address (Sydney Region)
13. IAM Role - appsync, AWSCodePipeline, codebuild, Cognito, event_handler, send_mail
14. CloudWatch logs - codebuild, lambda, ecs
15. CloudFormation Stack - msa-lab
