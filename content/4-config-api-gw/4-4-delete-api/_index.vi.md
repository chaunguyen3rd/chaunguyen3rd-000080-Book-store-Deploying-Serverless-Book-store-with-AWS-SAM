---
title : "Tạo DELETE API"
date :  2025-02-11
weight : 4
chapter : false
pre : " <b> 4.4. </b> "
---
1. Mở tệp **template.yaml** trong thư mục **fcj-book-shop**.

2. Thêm đoạn mã sau vào cuối tệp để tạo phương thức **DELETE**.
    - Đầu tiên, chúng ta cần làm mới để tạo phiên bản triển khai mới cho **POST** Api trong một vài bước tiếp theo. Bình luận khối **BookApiDeployment**.

      ```yml
      # BookApiDeployment:
      #   Type: AWS::ApiGateway::Deployment
      #   Properties:
      #     RestApiId: !Ref BookApi
      #   DependsOn:
      #     - BookApiGet

      BookApiStage:
        Type: AWS::ApiGateway::Stage
        Properties:
          RestApiId: !Ref BookApi
          StageName: !Ref stage
          # DeploymentId: !Ref BookApiDeployment
      ```

      ![CreateDeleteAPI](/images/temp/1/72.png?&width=90pc)
    - Chạy lệnh sau để triển khai SAM.

      ```bash
      sam build
      sam validate
      sam deploy
      ```

      ![CreateDeleteAPI](/images/temp/1/73.png?&width=90pc)
    - Tiếp theo, chúng ta sẽ tạo **BookApiDelete** và **BookApiDeleteInvokePermission**.

      ```yml
      BookApiDelete:
        Type: AWS::ApiGateway::Method
        Properties:
          HttpMethod: DELETE
          RestApiId: !Ref BookApi
          ResourceId: !Ref BookDeleteApiResource
          AuthorizationType: NONE
          Integration:
            Type: AWS_PROXY
            IntegrationHttpMethod: POST # For Lambda integrations, you must set the integration method to POST
            Uri: !Sub >-
              arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${BookDelete.Arn}/invocations
          MethodResponses:
            - StatusCode: "200"
              ResponseParameters:
                method.response.header.Access-Control-Allow-Origin: true
                method.response.header.Access-Control-Allow-Methods: true
                method.response.header.Access-Control-Allow-Headers: true

      BookApiDeleteOptions:
        Type: AWS::ApiGateway::Method
        Properties:
          HttpMethod: OPTIONS
          RestApiId: !Ref BookApi
          ResourceId: !Ref BookDeleteApiResource
          AuthorizationType: NONE
          Integration:
            Type: MOCK
            RequestTemplates:
              application/json: '{"statusCode": 200}'
            IntegrationResponses:
              - StatusCode: "200"
                ResponseParameters:
                  method.response.header.Access-Control-Allow-Origin: "'*'"
                  method.response.header.Access-Control-Allow-Methods: "'GET,POST,OPTIONS,DELETE'"
                  method.response.header.Access-Control-Allow-Headers: "'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token'"
          MethodResponses:
            - StatusCode: "200"
              ResponseParameters:
                method.response.header.Access-Control-Allow-Origin: true
                method.response.header.Access-Control-Allow-Methods: true
                method.response.header.Access-Control-Allow-Headers: true

      BookApiDeleteInvokePermission:
        Type: AWS::Lambda::Permission
        Properties:
          FunctionName: !Ref BookDelete
          Action: lambda:InvokeFunction
          Principal: apigateway.amazonaws.com
          SourceAccount: !Ref "AWS::AccountId"
      ```

      ![CreateDeleteAPI](/images/temp/1/79.png?&width=90pc)
    - Sau đó, chúng ta bỏ bình luận khối mã mà chúng ta đã bình luận ở trên.

      ```yml
      BookApiDeployment:
        Type: AWS::ApiGateway::Deployment
        Properties:
          RestApiId: !Ref BookApi
        DependsOn:
          - BookApiGet
          - BookApiCreate
          - BookApiDelete

      BookApiStage:
        Type: AWS::ApiGateway::Stage
        Properties:
          RestApiId: !Ref BookApi
          StageName: !Ref stage
          DeploymentId: !Ref BookApiDeployment
      ```

      ![CreateDeleteAPI](/images/temp/1/80.png?&width=90pc)

3. Chạy lệnh sau để triển khai SAM.

    ```bash
    sam build
    sam validate
    sam deploy
    ```

    ![CreateDeleteAPI](/images/temp/1/81.png?&width=90pc)

4. Mở [AWS API Gateway console](https://us-east-1.console.aws.amazon.com/apigateway/home?region=us-east-1).
    - Nhấp vào REST api **fcj-serverless-api**.
      ![CreateDeleteAPI](/images/temp/1/64.png?width=90pc)
    - Tại trang tài nguyên **fcj-serverless-api**.
      - Nhấp vào **Resources**.
      - Chọn **DELETE**.
      - Nhấp vào **Lambda integration** và kiểm tra hàm **book_delete**.
        ![CreateDeleteAPI](/images/temp/1/82.png?&width=90pc)
      - Nhấp vào **Stages**.
      - Chọn **DELETE**.
      - Sao chép và lưu **Invoke URL**.
        ![CreateDeleteAPI](/images/temp/1/83.png?&width=90pc)
