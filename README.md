# semantic-release 적용하기

## github action을 이용한 예제

- 커밋메시지를 일관적으로 작성하여 [시맨틱 버전](https://semver.org/)을 자동화 한다.
    - 일관적인 커밋메시지란 [AngularJS 커밋메시지 규칙](https://www.conventionalcommits.org)을 바탕으로한 컨벤셔널 커밋을 의미한다.

## 토큰 발급

- [github token 발급 링크](https://github.com/settings/tokens) 를 클릭해서 repo, workflow, write:packages 의 권한을 가진 github token을 발급받는다.
- 발급 받은 토큰을 복사해 semantic-release를 적용해줄 repository의 Settings > Secrets and variables > Actions 메뉴에 들어간다.

![토큰 설정 이미지1](https://github.com/user-attachments/assets/ac00b939-871e-4634-9780-c36da0b8d46d)
![토큰 설정 이미지2](https://github.com/user-attachments/assets/6be149b3-89b8-4aff-a575-f248e01de675)
![토큰 설정 이미지3](https://github.com/user-attachments/assets/6de2b5b1-5b2c-4b47-9696-d9e78b051da3)

- repository secret을 생성해주는데 이때 반드시 Name을 GH_TOKEN이라고 작성해야한다.
Secret에는 복사해준 토큰을 붙여넣어 준다.

## release.yml 작성

- github action에서 실행할 스크립트
```yaml
# main 브랜치에 push가 발생할 때만 실행
on:
  push:
    branches: [ main ]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      # 깃허브 저장소 체크아웃 (fetch-depth: 0은 전체 커밋 히스토리를 가져옴)
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
          
      # JDK 17 설정 (Spring Boot 3.x 기준)
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          
      # semantic-release는 Node.js 기반이므로 Node.js 설치
      - name: Setup Node.js
        uses: actions/setup-node@v3
          
      # semantic-release 관련 npm 패키지 설치
      # @semantic-release/git: 깃 커밋/태그 자동화
      # @semantic-release/changelog: CHANGELOG.md 자동 생성
      - name: Install dependencies
        run: npm install -g semantic-release @semantic-release/git @semantic-release/changelog
        
      # Gradle 캐시 설정으로 빌드 속도 향상
      - name: Cache Gradle packages
        uses: actions/cache@v3
        with:
          path: ~/.gradle/caches
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle') }}
          
      - name: Run semantic-release
        env:
          GH_TOKEN: ${{ secrets.GH_TOKEN }}
        run: npx semantic-release

```

## .releaserc.json 작성

semantic-release 설정을 담당하는데 아래와 같은 요소들을 설정해줄 수 있다.

- Git 저장소(URL, 배포할 브랜치, 태그 포맷)
- 사용할 플러그인 및 옵션 설정
- 실행 모드(디버그, dry run, local)

아래 예시를 실제 사용할 때는 주석을 삭제한다.
```json
{
  // main 브랜치에서만 릴리즈 수행
  "branches": ["main"],

  // 사용할 플러그인 설정
  "plugins": [
    // 커밋 메시지 분석하여 버전 결정
    "@semantic-release/commit-analyzer",

    // 릴리즈 노트 생성
    "@semantic-release/release-notes-generator",

    // CHANGELOG.md 파일 생성/업데이트
    "@semantic-release/changelog",

    // 변경된 파일 자동 커밋
    "@semantic-release/git",

    // GitHub 릴리즈 생성 및 자산 업로드
    [
      "@semantic-release/github",
      {
        "assets": [
          {
            // 빌드된 JAR 파일을 릴리즈에 첨부
            "path": "build/libs/*.jar",
            "label": "Spring Boot JAR"
          }
        ]
      }
    ]
  ]
}
```

## 플러그인들의 역할


### commit-analyzer

- feat: → minor 버전 증가 (1.0.0 → 1.1.0)
- fix: → patch 버전 증가 (1.0.0 → 1.0.1)
- BREAKING CHANGE → major 버전 증가 (1.0.0 → 2.0.0)


### release-notes-generator

- 커밋 메시지를 분석하여 릴리즈 노트 자동 생성
- 각 변경사항을 카테고리별로 정리 (Features, Bug Fixes 등)


### changelog

- CHANGELOG.md 파일에 버전 별 변경사항 기록
- 각 릴리즈마다 자동으로 업데이트


### git

- 버전 태그 생성
- CHANGELOG.md 변경사항 자동 커밋


### github

- GitHub Releases 페이지에 새 릴리즈 생성
- 빌드된 JAR 파일 자동 업로드

## 참고자료

### 공식
- [semantic-release 공식 깃허브](https://github.com/semantic-release/semantic-release)

### 블로그
- [semantic-release 을 이용하기 위한 예제](https://github.com/Karsei/semantic-release-demo)
- [Release note 생성, versioning 자동화 통합 도구 : Semantic Release](https://www.anyflow.net/sw-engineer/semantic-release)
- [[Project] 포트폴리오 제작기 -4. Sementic Release로 버저닝 및 CHANGELOG 관리하기](https://velog.io/@young_pallete/semantic-release)
- [semantic-release: 커밋 메세지로 버전 및 변경 로그 관리까지](https://coyo-hm.github.io/post/semantic-release)