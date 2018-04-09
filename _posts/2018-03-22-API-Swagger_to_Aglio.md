---
layout: post
title:  "API 문서화 for Spring 개발"
date:  2018-03-22
categories:
- REST
tags:
- API Blueprint
- apib
- Swagger
- Aglio
published: true
---
#### API 문서화 for Spring 개발
- 목적: Spring 개발 시 API 기능들에 대한 문서화에 대한 소개

- 일단 시작은 Swagger 부터 출발
  - 가장 흔히 사용하고 그렇기 때문에 다른 문서 변환을 위한 각종 컨버전 툴이나 라이브러리들이 많다
  - 이미 활용하고 있을 확률이 가장 높기 때문

- 하지만 Swagger는 개발자를 위한 참고문서는 되지만 조직 외부와 공유하는 문서(메뉴얼)라 보기는 어렵다
  - 내 생각에는 문서, 테스트 반반 개념
  - 'Try it out'의 위험성...
    - 쓸때 없는 서버 요청을 막으려면 "파라미터에 테스트 전용 예시 + Readonly"로 적용해주어야 하는 불편함
  - 문서적인 요소(주절주절 텍스트나 이미지)를 추가하기 어렵거나 소스를 더럽게(?) 함

- Swagger to Markdown with `swagger2markup`
  - 간단지만 간지는 조금 부족
  - https://github.com/Swagger2Markup/swagger2markup
  - Swagger2Markup은 Swagger JSON 또는 YAML 파일을 수작업으로 작성된 문서와 결합 할 수 있는 여러 AsciiDoc 또는 GitHub Flavored Markdown 문서로 변환합니다.
  - Gradle 플러그인: `swagger2markup-gradle-plugin`
  - Maven 플러그인: `swagger2markup-maven-plugin`

- Swagger to API Blueprint with `swagger2blueprint`
  - 간지를 위해 효율을 조금 내려놓는다면
  - API Blueprint: https://apiblueprint.org/
  - 'swagger2blueprint'의 주의 사항: Swagger의 모든 정보가 온전하게 변환되지 않는다. (특히 Swagger의 Example 항목) 하지만 공백부터 작성하는 것보다는 편의를 주는데 의미를 찾아봅니다.

    ```bash
    [설치]
    npm install -g swagger2blueprint

    [실행: Swagger JSON를 변환]
    /usr/local/bin/swagger2blueprint http://[Swagger Server]/v2/api-docs api.apib

    [`Additional properties not allowed: allowEmptyValue` 에러 발생 시]
    wget http://[Swagger Server]/v2/api-docs && find ./ -name "api-docs" -exec perl -pi -e 's/,"allowEmptyValue":false//g' {} \;
    /usr/local/bin/swagger2blueprint api-docs api.apib
    ```

  - API Blueprint 편집
    - 메뉴얼 문서의 성격을 추가한다면 편집기를 통해 설명 보완
    - https://apiblueprint.org/tools.html
      - Apiary
      - vscode-apielements
      - (나는 이거 씀) Atom editor preview (with `aglio`)
      - 등등

- API Blueprint to HTML with `aglio`
  ```bash
  [설치]
  npm install -g aglio

  [실행]
  aglio -i api.apib -o api.html

  [실행: 다른 테마 사용]
  aglio -i api.apib --theme-variables slate -o api.html
  ```

- Github Page
  - Repository의 'GitHub Pages' 설정
    - 해당 경로에 최종 완성된 HTML 또는 Markdown 파일을 Commit
    - https://[Github Pages Domain]/[User]/[Repository]/[File]
    - Markdown 일 경우 파일명을 'README.md'로 저장하면 파일명을 URL에 넣을 필요 없다.