# Lots-of-minifb
Lots of minifb implementation

동일한 백엔드 어플리케이션(조그마한 페이스북 clone)에 대해, 기술스택을 바꿔가며 수많은 구현체를 만들어 나가는 프로젝트([realworld](https://github.com/gothinkster/realworld)처럼)

## 전제조건
- 별도의 인프라 없이, Amazon EC2와 같은 compute cloud에서 자체적으로 구성한다고 가정 : ex) 파일 관리를 위해 S3를 별도로 사용하지 않음
- 모든 API는 테스트되어야 함(view function에 한해서는, 되도록 테스트 커버리지를 100%로 유지)
- 테스트 프레임워크와 서드파티(CI, Coverage, ...)들은 기술스택의 경우의 수에서 제외(Flask + SQLAlchemy + pytest와 Flask + SQLAlchemy + unittest를 별도의 스택으로 생각하지 않음). 따라서 취향 따라 선택하되, 다양성은 존재해야 함
- 가이드라인에 맞추어 API만 제공된다면, 내부 로직은 자유롭게 구성(동일한 API에 대해 프로젝트 A는 group by를 쓰고, 프로젝트 B는 distinct 쿼리를 쓰더라도 자유성을 인정). 하지만 단지 API를 찍어내는 형태가 아닌, 개발 과정에서 자신도 성장해야 하며 나름대로 많이 고민해 주어야 함.(예제들을 보는 입장에서 새로운 아이디어를 얻어갈 수 있도록)
- 기술스택의 범주는 언어 / 프레임워크 / 데이터베이스 / DB 클라이언트 라이브러리 / 캐시(optional) / API 아키텍처(REST, GraphQL, ...) / 사용자 인증
- 부가 기술의 범주는 테스트 프레임워크 / 의존성 관리 / 로깅 / API 문서 / CI / Coverage / Code Qualitifier. API 문서부터는 모두 optional
- 백엔드 엔지니어링적인 측면에서의 고민이 녹아들어 있으면 좋음(싱글 스레드로 동작하는 redis에 대해 실행 시간을 많이 잡아먹는 operation을 하지 않거나, 데이터베이스 부하에 대한 유연성 고려 등)

## Model(Validation 포함)
### user
- ID(string, PK, length 4~50)
- 비밀번호(string, pbkdf2_hamc hashed : like Werkzeug's generate_password_hash, length 10~100)
- 이름(string, length 2~16)
- 별명(string, nullable, length 1~30)
- 소개(string, nullable, length 1~85)
- created_at(timestamp, default now)

### post
- owner(References user)
- 내용(string, length 1~3000)
- created_at(timestamp, default now)

### reaction_post
- 사용자(References user)
- 글(References post)
- 리액션 타입(int, 1~5)
- 수정됨 여부(bool, default false)
- created_at(timestamp, default now)

### comment
- 사용자(References user)
- 글(References post)
- 내용(string, length 1~1000)
- 수정됨 여부(bool, default false)
- created_at(timestamp, default now)

### reaction_comment
- 사용자(References user)
- 댓글(References comment)
- 리액션 타입(int, 1~5)
- created_at(timestamp, default now)

### notification
- 사용자(References user)
- 알림 타입(int, 1~2, 1: 내 글에 댓글을 남김, 2: 내 글에 좋아요를 남김)
- 알림을 일으킨 사용자(References user)
- 연걸된 글(References post)
- 읽음 여부(bool, default false)
- created_at(timestamp, default now)

## API
### Frontend API
#### 계정
- ID 중복체크
- 로그인
- 회원가입
- 임시 비밀번호 발급
- 비밀번호 변경
- 별명 변경
- 소개 변경
- 재량에 따라, 인증 관련 API(JWT token refresh 등)

#### 글
- 게시글 업로드
- 게시글 수정
- 게시글 삭제
- 게시글 목록 조회(pagination)

#### 타임라인
- 내 타임라인
- 다른 사용자의 타임라인

### Back Office API
