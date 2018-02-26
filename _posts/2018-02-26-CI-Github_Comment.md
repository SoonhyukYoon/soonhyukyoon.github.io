---
layout: post
title:  "Github Comment"
date:  2018-02-26
categories:
- CI
tags:
- Comment
published: true
---
## Example
- Refactor subsystem X for readability
- Update getting started documentation
- Remove deprecated methods
- Release version 1.0.0

## Bad
- Fixed bug with Y
- Changing behavior of X
- More fixes for broken stuff
- Sweet new API methods

## Basic Rule
- 50 문자 이내로 간략히 작성
- 마침표 지양
- 대문자로 시작
- 무엇(What)을 했는지 표현
  - Bad Case1) Fix
  - Bad Case2) Refactor
- 관련 이슈가 있을 경우 번호 언급. github의 Closing issues via commit messages을 사용
  - Refactor subsystem X for readability #1010
  - Refactor subsystem X for readability. Closes #1010

## 참고
- http://www.slideshare.net/TarinGamberini/commit-messages-goodpractices
- http://chris.beams.io/posts/git-commit/
