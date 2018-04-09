---
layout: post
title:  "NPM 사설 저장소 of Nexus"
date:  2018-01-15
categories:
- CI
tags:
- NPM
- Nexus

published: true
---
#### NPM 사설 저장소 of Nexus
##### Sonatype Nexus
- 오픈 소스 기반 응용 프로그램 보관 리소스(빌드 결과물) 저장소
- 저장소 관리자로, 다양한 Format(Maven, NPM, pypi 등)의 사설 저장소를 만들 수 있으며 외부(Remote) 메인 저장소를 Cache할 수 있는 기능 또한 제공하여 저장소를 관리할 수 있도록 도와주는 관리자 도구: Web & API
- NPM Repository 구성 가능

##### Purpose
- 내부 공통 Front/Backend 라이브러리 관리 및 응용 컴포넌트 간 활용
- 잘 정리된 Components 공유와 개발인원 전파

##### Repository
- npm-public
  - URL: http://*NEXUS_HOST*/repository/npm-public
  - Type: Group (NPM Registry + `npm-release`)
- npm-release
  - URL: http://*NEXUS_HOST*/repository/npm-release
  - Type: Private (개발자 Node.js Archives)

##### Deploy application
- 개발 대상 node.js 프로그램 루트에 아래와 같은 작업 추가
- `package.json` 패키지 명세 추가
```json
{
  "name": "LIB_NAME",
  "version": "VERSION",
  "description": "DESCRIPTION",
  "author": {
    "name": "AUTHOR_NAME",
    "email": "EMAIL"
  },
  "private": false,

  "...":"이하생략"
}
```

- `.npmignore` 저장소 업로드 예외 파일 목록
```bash
.DS_Store
.git
.*.swp
.npmrc
.svn
npm-debug.log
.idea
*.iml

등등...
```

- 계정 등록
```bash
~$ npm adduser --registry http://NEXUS_HOST/repository/npm-release/
Username: [계정 입력]
Password: [패스워드 입력]
Email: (this IS public) [이메일 입력]
Logged in as [계정] on http://NEXUS_HOST/repository/npm-release/.
```

- `.npmrc` 파일에 'authToken' 확인
```bash
//NEXUS_HOST/repository/npm-release/:_authToken=TOKEN
```
- Publish (`package.json` 경로에서)
```bash
$ npm publish --registry http://NEXUS_HOST/repository/npm-release/
```

##### Usage
- npm Registry 변경 (`npm-public` of Nexus)
```bash
$ npm config set registry http://NEXUS_HOST/repository/npm-public/
```

- ~/.npmrc
```bash
registry=http://NEXUS_HOST/repository/npm-public/
```

- Tip. `package.json` - 등록된 의존 라이브러리 경로 직접 입력
  - 'NPM Registry'에 등록된 라이브러리와 이름 중복 가능성이 있다면 아래와 같은 방식으로 대상 URL을 직접 지정 가능
  ```json
  {
    "dependencies": {
      "라이브러리이름": "http://NEXUS_HOST/repository/npm-release/주소/-/파일-버전.tgz"
    }
  }
  ``` 

