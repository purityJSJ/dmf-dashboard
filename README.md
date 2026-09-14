# 원료의약품등록(DMF) 현황 대시보드

식품의약품안전처 「원료의약품등록(DMF)현황」 공공데이터 API를 매일 아침 9시(KST)에
GitHub Actions로 수집해서, 정적 웹페이지(대시보드 + 목록)로 보여주는 프로젝트입니다.

## 구성

```
.github/workflows/fetch-dmf.yml   # 매일 09:00 KST 실행되는 크론 워크플로우
scripts/fetch-dmf.mjs             # API 호출 + 데이터 가공 스크립트
data/latest.json                  # 원본 수집 데이터 (Actions가 자동 갱신)
docs/index.html                   # 정적 대시보드 페이지
docs/data/latest.json             # 웹페이지가 실제로 읽는 데이터 (Actions가 자동 갱신)
```

## 처음 설정하는 방법

1. **리포지토리 생성**: 이 폴더 전체를 GitHub 리포지토리로 올려주세요.

2. **API 인증키 등록 (GitHub Secret)**
   - 리포지토리 → `Settings` → `Secrets and variables` → `Actions` → `New repository secret`
   - Name: `DMF_API_KEY`
   - Value: 공공데이터포털에서 발급받은 인증키 (URL 인코딩된 형태 그대로 붙여넣으면 됩니다)
   - ⚠️ 대화 중에 공유해주신 인증키는 이미 노출된 상태이니, 공공데이터포털에서
     **재발급(마이페이지 → 활용신청 현황 → 인증키 재발급)** 받은 새 키를 등록해주세요.

3. **GitHub Pages 활성화**
   - `Settings` → `Pages` → Source: `Deploy from a branch`
   - Branch: `main`, 폴더: `/docs` 선택 후 저장
   - 몇 분 후 `https://<사용자명>.github.io/<리포지토리명>/` 에서 대시보드 확인 가능

4. **워크플로우 최초 실행**
   - `Actions` 탭 → `Fetch DMF data` → `Run workflow` 클릭해서 수동으로 한 번 실행
   - 정상 실행되면 `data/latest.json`, `docs/data/latest.json`이 커밋되고, 페이지에 데이터가 표시됩니다.
   - 이후에는 매일 09:00 KST(=00:00 UTC)에 자동 실행됩니다.

## 로컬에서 테스트하기

```bash
export DMF_API_KEY="발급받은_인증키_URL인코딩된_그대로"
node scripts/fetch-dmf.mjs
```

성공하면 `data/latest.json`이 생성됩니다. 이를 `docs/data/latest.json`으로 복사한 뒤
`docs/index.html`을 브라우저로 열면 로컬에서도 확인할 수 있습니다 (정적 파일이라 별도 서버 없이도
대부분 브라우저에서 fetch가 동작하지만, CORS 문제가 있으면 `npx serve docs` 등으로 간단히 로컬 서버를
띄워서 확인하세요).

## 커스터마이징 힌트

- **필터 파라미터 추가**: `scripts/fetch-dmf.mjs`의 `fetchPage()`에서 `entp_name`(업체명),
  `ingr_kor_name`(성분명) 파라미터를 추가하면 특정 업체/성분만 수집할 수 있습니다.
- **수집 시간 변경**: `.github/workflows/fetch-dmf.yml`의 `cron` 값을 수정하세요.
  (cron은 UTC 기준이라 KST 09:00 = UTC 00:00 입니다.)
- **데이터 보관**: 지금은 최신 데이터만 덮어쓰지만, 날짜별 스냅샷을 남기고 싶다면
  `data/YYYY-MM-DD.json` 형태로 추가 저장하도록 스크립트를 확장하면 됩니다.

## 참고

- API 상세: https://www.data.go.kr/data/15057075/openapi.do
- 요청 URL: `https://apis.data.go.kr/1471000/MdcDmfInfoService01/getMdcDmfList01`
