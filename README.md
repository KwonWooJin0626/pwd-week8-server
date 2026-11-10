# Ajou Campus Foodmap — 8주차 서버

[공식 실습](https://github.com/ajou-hyunseok-oh/pwd-week8-server/blob/main/README.md)에 맞춘 Express·TypeScript·Supabase 서버입니다. 기존 7주차 서버와 별도로 준비했습니다.

## 로컬 실행

Node.js 24.x를 사용합니다. `.env`의 URL·포트·공개 키는 현재 클라이언트와 맞췄습니다. 같은 프로젝트의 서버용 `SUPABASE_SECRET_KEY`도 로컬 파일에 입력되었습니다. 이 값은 프런트엔드나 GitHub에 넣지 않습니다.

```powershell
npm ci --include=dev
npm run dev
```

- 기본 서버: `http://localhost:4000`
- 상태: `http://localhost:4000/health`
- 클라이언트: `http://127.0.0.1:5173`

## DB 준비

Supabase 프로젝트 `joovhrxjefwmydoaezrx`에 `supabase/schema.sql`, 이어서 `supabase/seed.sql`을 적용했습니다. schema는 프로필·맛집·제보 테이블, Auth 프로필 트리거, 관리자 역할과 제보 승인 RPC, 테이블 접근 권한을 준비합니다. seed는 맛집 목록이 비었을 때만 예시 데이터를 넣습니다. 현재 기본 맛집 3건이 등록되어 있으며 SQL을 다시 수동 실행할 필요는 없습니다.

```powershell
npm run db:check
```

지정한 본인 이메일의 최초 관리자 계정은 생성·활성화했습니다. 임시 로그인 정보는 Git에서 제외한 `.env.admin.local`에 저장했습니다. 로그인 후 내 정보에서 임시 비밀번호를 변경합니다. 이후 다른 가입 계정의 역할을 바꾸려면 아래 명령을 사용합니다.

```powershell
npm run change-user-type -- 본인의가입이메일 admin
```

## 구현한 API

| 경로 | 용도 |
| --- | --- |
| `/health` | 프로세스 상태 |
| `/api/restaurants`, `/popular`, `/:id` | 공개 목록·인기·상세 조회 |
| `/api/restaurants`, `/:id` | 관리자 생성·수정·삭제 |
| `/api/auth/me`, `/providers` | 사용자 확인·공급자 준비 상태 |
| `/api/users/profile`, `/account` | 프로필 변경·계정 비활성화 |
| `/api/users/all`, `/:id/type` | 관리자 사용자 조회·역할 변경 |
| `/api/submissions`, `/:id`, `/:id/approve` | 제보 작성·관리·승인 |

## 검사

```powershell
npm run typecheck
npm test
npm run build
npm run auth:check
npm run test:live
```

로컬 API·서비스·공급자 테스트 58개와 PostgreSQL 동작 테스트 10개가 통과했습니다. `test:live`도 실제 Supabase에서 통과했습니다. 이 검사는 이번 검사 전용 계정과 데이터를 생성한 뒤 정리하며 기존 맛집 3건이 보존됐는지 확인합니다.

`/health` 성공은 서버 프로세스 확인이며 DB·로그인 준비 완료와는 별도입니다.

## Google·네이버 설정

공급자 콜백은 `https://joovhrxjefwmydoaezrx.supabase.co/auth/v1/callback`입니다. 앱으로 돌아오는 주소는 `http://127.0.0.1:5173/auth/callback`입니다.

- Supabase URL Configuration에서 Site URL과 Redirect URLs를 설정합니다.
- Google OAuth 웹 클라이언트의 origin은 클라이언트 origin, redirect URI는 위 Supabase 공급자 콜백입니다. Client ID·Secret은 Supabase Google 공급자 설정에 등록합니다.
- 네이버 앱의 서비스 URL은 클라이언트 주소, Callback URL은 위 Supabase 공급자 콜백입니다. 테스터도 등록합니다.
- 네이버 Client ID·Secret을 서버 `.env`에 넣은 후 `npm run auth:setup-naver`로 OIDC 공급자를 등록합니다. 키가 없는 상태에서는 등록하지 않습니다.
- `npm run auth:check` 후 실제 공급자 계정으로 로그인·취소·새로고침·로그아웃을 확인합니다.

## Render 배포

`render.yaml`로 신규 무료 Node 웹 서비스를 배포했습니다. 주소는 `https://pwd-week8-server-kwonwoojin-hpek.onrender.com`이며 `/health`, 공개 조회, 실제 관리자 로그인 후 `/api/auth/me`·사용자 목록의 200 응답을 확인했습니다. Render에 서버 환경변수를 적용했으며 `CLIENT_URL`에는 실제 Vercel origin과 로컬 origin을 등록했습니다.

실제 HTTPS API 주소를 클라이언트 Vercel 환경변수 `VITE_API_BASE_URL`에 등록했습니다. 클라이언트 공개 운영 주소는 `https://pwd-week8-client-ashy.vercel.app`입니다. Supabase와 외부 공급자의 주소 설정에도 이 Vercel 주소를 추가해야 합니다.

서버 키·DB 준비·실제 Auth/API 통합 검증·최초 관리자 지정·Render 배포·Vercel 배포는 완료했습니다. Supabase URL Configuration, Google·네이버 앱 발급·공급자 설정과 실제 소셜 로그인은 남아 있습니다. GitHub·배포 진행 결과는 클라이언트의 `SETUP_STATUS.md`, 외부 설정 단계는 `OAUTH_SETUP.md`에 기록합니다.

참고: [Render Blueprint 규격](https://render.com/docs/blueprint-spec), [Supabase Google 로그인](https://supabase.com/docs/guides/auth/social-login/auth-google), [Supabase Redirect URLs](https://supabase.com/docs/guides/auth/redirect-urls).
