emozzal.com(img)
세상의 모든 이모티콘(sup : 슬로건img)

# 서버 이용 방안

* EC2: 백엔드 서버
* RDS: 메타데이터 (사용자 정보, 이모티콘/짤방 정보, 태그 등) - RDS 설정 완료(2025.05.08)
* S3: 이모티콘 이미지 (작은 용량) - 기존의 프론트엔드 S3 버킷 삭제 및 새 버킷 생성, CORS 설정 완료(2025.05.08)
* Cloudinary: 짤방 이미지 (큰 용량)
* Vercel: 프론트엔드

- AWS RDS는 관계형 데이터베이스 서비스로, 구조화된 데이터를 저장하고 처리하는 데 최적화되어 있음. 반면 S3는 이미지, 오디오, 비디오 파일 등 다양한 형식의 객체를 저장하는 데 적합한 오브젝트 스토리지임.

## 실제 구현 방식

* RDS에는 이미지 메타데이터만 저장:
** 이미지 이름, 태그, 카테고리, 업로더 정보 등
** 이미지 URL (S3 또는 Cloudinary에 저장된 실제 파일 위치)

* S3/Cloudinary에는 실제 이미지 파일 저장:
** S3: 작은 크기의 이모티콘 (5GB 내외)
** Cloudinary: 큰 크기의 짤방 (25GB까지 무료)

* 이미지 업로드/접근 흐름:
** 사용자가 이미지 업로드 → 백엔드(EC2)에서 처리
** 이미지는 S3/Cloudinary에 저장
** 메타데이터 + URL은 RDS에 저장
** 이미지 조회 시 RDS에서 URL 조회 → S3/Cloudinary에서 이미지 로드



# 이모티콘 플랫폼 설계 및 운영 방안

## 1. 플랫폼의 핵심 기능

### (1) 타 사이트의 각종 이모지를 종합적으로 정리하여 필터를 통해 검색 및 다운로드 / 구매연결

* 사이트별 필터링 구현
* 각 필터링별 카테고리 구현
* 검색된 무료 이모티콘 미리보기 제공(워터마크) / 즉각적 다운로드 제공 및 필요시 URL 제공
* 검색된 유료 이모티콘 미리보기 제공(워터마크) / URL 제공
* 필터링 : 사이트 - 유/무료
** 해당 사이트나 커뮤니티에 알맞은 카테고리나 필터링 각각 추가
** 필터링 제외한 검색기능 추가 - 띄어쓰기나 오탈자 보완기능 추가
** 텍스트 검색, 카테고리 검색, 태그 검색

### (2) 창작물 업로드 및 관리

* 창작자가 자신이 제작한 이모티콘이나 짤방을 업로드.
* 업로드 시 창작자가 이미지 URL과 미리보기 이미지를 함께 제출.
* 플랫폼에서 미리보기 이미지에 자동으로 워터마크 추가.


### (3) 창작물 활용 방식

* 창작자가 자신이 활동하는 커뮤니티(예: 디시인사이드, 인벤)에 창작물을 업로드.
* 창작마당에는 창작자가 올린 창작물을 홍보하는 느낌으로 URL과 함께 등록.
* URL을 통해 원본 창작물로 직접 연결.

### (4) 신고 시스템

* 신고 버튼을 통해 문제가 되는 창작물을 신고 가능.
* 신고 시:

  1. 신고 내용을 입력.
  2. 관리자인 운영자와 창작자에게 쪽지 형태로 신고 내용 전달.
  3. 운영자는 신고 내용을 확인 후 필요한 조치를 진행.

### (4) 사용자 정책 명시

* 창작마당 이용 약관을 공지사항으로 게시.
* 약관에 저작권 표시 삭제, 무단 재배포 금지 등을 명시.
* 단순 공지사항으로 끝내지 말고, 사용자 동의 과정을 반드시 포함해야 함.

---

## 2. 부가 기능

### (1) 기존의 이모티콘

* 이모티콘이 아닌 짤방이 주류인 커뮤니티도 있으므로 짤방역시 메인
* 

### (2) 창작마당 - OAuth로 로그인 필요

* 댓글 및 피드백 시스템

### (3) 트래픽 유도 요소

* 추천 이모티콘 : 실시간 인기 이모티콘 랭킹
* 이모티콘 제작 튜토리얼 : 사용자가 쉽게 참여할 수 있도록 제작 가이드 제공.
* 태그 검색(혹은 필터)이나 한눈에 여러가지를 미리 볼 수 있는 기능을 만들어 검색 UI/UX강화

---

## 3. 데이터 구조 설계

### (1) 주요 테이블 구조

* **Emoticons** (이모티콘 및 짤방 정보 저장):

  * `id`, `name`, `url`, `preview_url`, `type` (무료/유료), `tags`, `created_at`, `creator_id`
* **Creators** (창작자 정보):

  * `id`, `nickname`, `email`, `bio`
* **Reports** (신고 내용 저장):

  * `id`, `emoticon_id`, `reporter_id`, `description`, `created_at`
* **Categories** (카테고리 정보):

  * `id`, `name`, `description`
* **Emoticon\_Category** (다대다 관계):

  * `emoticon_id`, `category_id`

---

## 4. 기능 상세 설계

### (1) 이모티콘 업로드 페이지

* 창작자가 URL과 미리보기 이미지를 입력할 수 있는 업로드 폼 제공.
* 미리보기 이미지는 서버에서 워터마크가 자동 추가된 상태로 저장.

### (2) 이모티콘 관리

* 창작자는 자신의 업로드 이모티콘 목록을 확인하고 수정/삭제 가능.
* 관리자는 모든 이모티콘을 관리(수정, 삭제, 신고 처리).

### (3) 신고 처리

* 신고가 접수되면 운영자와 창작자에게 알림 쪽지 발송.
* 운영자는 신고 내역을 검토하고 적절한 조치를 결정.

---

## 5. 백엔드 설계 (Spring Boot)

### (1) 주요 API 엔드포인트

* **이모티콘**:

  * `GET /api/emoticons` - 전체 이모티콘 조회
  * `POST /api/emoticons` - 이모티콘 업로드
  * `DELETE /api/emoticons/{id}` - 이모티콘 삭제
* **신고**:

  * `POST /api/reports` - 신고 접수
  * `GET /api/reports` - 신고 내역 조회 (관리자 전용)

### (2) 기타

* Swagger를 활용해 API 문서 자동 생성.
* JWT를 사용한 사용자 인증 및 권한 관리.

---

## 6. 프론트엔드 설계 (React)

### (1) 주요 컴포넌트

* **UploadForm**: 창작자가 이모티콘을 업로드하는 폼.
* **EmoticonList**: 전체 이모티콘을 카테고리별로 표시.
* **ReportForm**: 신고 내용을 작성하는 폼.

### (2) 상태 관리

* Redux 또는 Context API로 글로벌 상태 관리.
* Axios를 사용해 백엔드와 통신.

---

## 7. 데이터 준비 및 운영 방안

### (1) 데이터 준비

* 크롤링 도구를 사용해 샘플 데이터를 수집(Python + BeautifulSoup).
* 수집 데이터:

  * 이름, URL, 미리보기 URL, 태그, 카테고리 등.

### (2) 운영

* 사용자 피드백을 통해 플랫폼 개선.
* 정기적으로 업로드된 콘텐츠를 검토하고 불법 콘텐츠 제거.

---

## 8. 구현 우선순위

1. 데이터베이스 설계 및 구축
2. 백엔드 API 개발
3. 프론트엔드 UI 및 기능 구현
4. 신고 시스템 및 사용자 정책 완성
5. 테스트 및 배포

## 9. 추가 고려사항

### (1) 사이트/커뮤니티별 특성 파악

* 주 잠재적 주 고객층의 커뮤니티에서 이모티콘 시스템이 어떻게 작동하는 지 명확한 확인 전제
* 특히 이모티콘을 따로 샵 카테고리로 지정하지 않은 커뮤니티등에 대한 대응 및 데이터 구축방안이 키포인트
* 저작권 문제가 가장 중요, 창작자에게 허가 등의 어려움 예상
* 디시인사이드 사이트에 관해선 다수의 갤러리가 있으므로 순위와 성향을 파악하는 것이 전제
* 디시인사이드 사이트는 이모티콘 코드 형태로 작성
** 무료로 배포된 GIF, PNG 이미지 파일을 텍스트 코드로 변환하여 사용 > 메커니즘 파악
* 인벤은 이모티콘 보다는 짤방 문화가 강한 편
* 카카오톡이나 네이버는 스토어 URL로 편의성 예상, 저작권에 대한 적극적인 공부 필요

* 대형 사이트 및 커뮤니티 파악
** 카카오톡, 네이버, 디시인사이드, 클리앙, 웃긴대학, 루리웹, 다음카페(야옹이카페, 쭉빵카페, MLB카페, 화력지원카페, 맘스홀릭)
** 네이버카페(소울드레서, 초코맘, 레몬테라스, 오늘의집 카페, 헝그리앱 게임 커뮤니티)
** 네이버 블로그 기반 커뮤니티(윰갤 블로그, 덕후 커뮤니티 블로그)
** 뽐뿌, MLBPARK, 

### (2) 저작권 문제

#### 창작마당 콘텐츠에 대해 저작권 보호에 대한 적극적인 방어요구

* 창작자가 업로드한 콘텐츠는 별도의 이용 약관을 통해 저작권을 보호해야 함.

#### 약관 필수 항목

* 창작물에 대한 권리 귀속
* 플랫폼 내에서의 사용 허용 범위
* 사용자의 비상업적 이용 허용 여부

#### 무료 배포 이모티콘의 비상업적 사용 조건과 영리적 사용으로의 변질 가능성

* 문제 분석

** 무료 배포 이모티콘은 주로 "비상업적 사용만 가능"이라는 조건을 내걸고 배포됨
** 초기에는 영리 목적이 없어도, 사이트가 성장하여 광고 수익(배너 광고 등)이 발생하면, 제작자의 관점에서 사이트에서 해당 이모티콘을 사용하는 것이 영리적 활동으로 간주될 가능성이 높음
** 특히, 사이트 운영으로 발생하는 수익이 이모티콘을 통해 얻은 트래픽에 크게 의존할 수 밖에 없는 구조

* 대응책

** 명확한 허가 : 배포자에게 개별적으로 연락하여, 해당 이모티콘을 광고 수익이 발생하는 플랫폼에서도 사용 가능하다는 허락을 문서화(이메일 등)
** 가능하면 제작자와 라이선스 무료계약 체결
** 비상업적 이모티콘과 상업적 사용 분리: 무료 배포 이모티콘은 비상업적 카테고리로 분리하고, 광고가 없는 페이지에서만 노출, 상업적 사용을 허용한 이모티콘만 별도로 모아 배너 광고가 있는 페이지에 활용. (아마 카테고리 구조상 난항예상)
** 제작자 등록 시스템 추가 : 이모티콘 제작자가 본인의 이모티콘을 플랫폼에 직접 업로드하고, 사용 조건(비상업적, 상업적 허용 등)을 설정할 수 있도록 시스템화. (이 방식은 플랫폼이 성장하며 자연스럽게 저작권 문제를 완화할 수 있음)

####  유료 이모티콘 미리보기 목록 제공

* 저작권 침해 가능성
** 미리보기 목록은 일반적으로 저작권 침해가 아니라고 판단될 가능성이 큼(해당 이미지는 이미 카카오톡, 네이버 등에서 구매 전에 누구나 볼 수 있는 상태로 공개되어 있음. 사용자가 구매 전 확인을 위해 제공된 정보를 그대로 노출하는 것이기 때문.)
** 하지만 해당 미리보기 이미지를 가져올 때, 원본 이미지를 직접 복제하거나 저장하지 않고, 외부 URL로 동적 로드(서버 요청)하여 표시해야 함.
** 예를 들어, 카카오톡 이모티콘 스토어의 이미지 URL을 통해 데이터를 가져오고 표시하는 방식은 저작권 침해를 피할 수 있음.
** 따라서 이미지 소스 명시 : 미리보기 이미지를 표시할 때, 출처(URL)을 명확히 기재필수. (출처 : 카카오 이모티콘 스토어 와 같이 표시)
** 동적 로드 방식 사용 : 이미지를 외부 API나 URL을 통해 로드하고, 자체적으로 복제하거나 저장하지 않음.
** 장기적으로 서비스가 성장할 경우, 변호사나 저작권 전문 컨설턴트 자문을 통해 리스크를 최소화.

#### 창작마당 콘텐츠 약관 설정

* 문제 분석

** 창작마당에 업로드된 콘텐츠는 저작권 분쟁의 소지가 있으므로 명확한 이용 약관이 필요, 단순히 공지사항으로 약관을 게시하는 것은 법적 효력이 제한될 수 있음.

* 올바른 약관 설정 방법

** 이용 약관 동의 절차 추가: 사용자가 창작마당에 콘텐츠를 업로드하거나 다운로드하기 전에, 이용 약관에 동의하도록 설정.
** 약관 동의 버튼(체크박스)을 클릭해야 업로드/다운로드가 가능하도록 구현. (단순 공지사항으로 끝내지 않고, 사용자 동의 과정을 반드시 포함하도록)

* 약관 필수 항목

1) 저작권 귀속
* 업로드한 콘텐츠의 저작권은 창작자에게 귀속되며, 플랫폼은 해당 콘텐츠를 비영리/영리 목적으로 표시, 배포할 수 있는 라이선스를 가짐.

2) 저작물의 책임
* 업로드된 콘텐츠가 타인의 저작권을 침해하지 않아야 하며, 침해 시 모든 책임은 창작자에게 있음.

3) 플랫폼의 면책
* 플랫폼은 사용자 간 저작권 분쟁에 대해 책임을 지지 않음을 명시.

4) 약관 공지 및 보관
* 약관 내용을 공지사항에 게시하되, 업로드/다운로드 페이지에도 명시적으로 표시.
* 이용자가 언제든 약관을 확인할 수 있도록 접근성을 보장.

5) 약관 예시:
1. 창작자는 자신이 업로드하는 콘텐츠의 모든 저작권 및 관련 권리를 보유하며, 이를 제3자의 권리를 침해하지 않도록 보장해야 합니다.
2. 플랫폼은 업로드된 콘텐츠를 서비스 운영 및 홍보 목적으로 사용하거나 표시할 권한을 가집니다.
3. 창작자가 업로드한 콘텐츠로 인해 발생하는 모든 법적 분쟁 및 책임은 창작자가 전적으로 부담합니다.
4. 창작마당에 업로드된 모든 콘텐츠는 비상업적 목적으로만 사용 가능합니다.

* smallDesciption
비상업적 사용 조건에 대비해 초기부터 제작자와 명확히 협의하고, 플랫폼이 성장하며 영리적 사용으로 전환되는 경우를 대비한 절차 마련.











#1. 데이터 정리 기본 계획
##1) 이모티콘/짤방 데이터
##2) 메타데이터
- 이름 (임티 이름, 짤방 제목 등)
- 출처 (해당 URL, 저작권 관련 정보)
- 카테고리 (유머, 밈, 감정 등)
- 사이트 (카카오, 네이버, 각종 커뮤니티)
- 태그 (슬픔, 웃음, 고양이 등)
- 업로드 날짜/시간
- 제작자/업로더 정보
##3) 테이블 구분
* 공통 데이터
- EmojiCommon: 모든 이모티콘 및 짤방의 공통 속성을 관리.
- ZzalCommon: 짤방의 공통 속성을 관리.
- Uploader: 업로더 정보 관리.

* 유형별 데이터
- FreeEmoji: 무료 이모티콘의 데이터 관리.
- PaidEmoji: 유료 이모티콘의 데이터 관리.
- CreatedEmoji: 창작마당 이모티콘 데이터 관리.
- ZzalCategories: 짤방의 카테고리 데이터 관리.

* 추가기능 데이터
- EmojiCategories: 이모티콘 카테고리 데이터 관리.
- Report: 신고 데이터를 관리.

#2. 테이블 설계 및 관계 설정
##1) 공통 테이블 : EmojiCommon 테이블
- 모든 이모티콘의 공통 속성을 관리하는 테이블

CREATE TABLE EmojiCommon (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL, -- 이모티콘 이름
    category_id INT, -- 카테고리 ID (카테고리 테이블과 연동)
    tags VARCHAR(255), -- 태그 (콤마로 구분된 문자열)
    uploader_id INT NOT NULL, -- 업로더 ID (Uploader 테이블과 연동)
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP, -- 업로드 날짜/시간
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES EmojiCategories(id),
    FOREIGN KEY (uploader_id) REFERENCES Uploader(id)
);

##2) 무료 이모티콘 : FreeEmoji 테이블
- 무료 이모티콘의 URL 정보 관리하는 테이블

CREATE TABLE FreeEmoji (
    id INT AUTO_INCREMENT PRIMARY KEY,
    emoji_common_id INT NOT NULL, -- EmojiCommon과 연동
    url TEXT NOT NULL, -- 이미지 URL
    FOREIGN KEY (emoji_common_id) REFERENCES EmojiCommon(id) ON DELETE CASCADE
);

##3) 유료 이모티콘 : PaidEmoji 테이블
- 유료 이모티콘의 구매 페이지 URL 및 가격 정보를 관리하는 테이블

CREATE TABLE PaidEmoji (
    id INT AUTO_INCREMENT PRIMARY KEY,
    emoji_common_id INT NOT NULL, -- EmojiCommon과 연동
    purchase_url TEXT NOT NULL, -- 구매 페이지 URL
    price DECIMAL(10, 2) NOT NULL, -- 가격
    FOREIGN KEY (emoji_common_id) REFERENCES EmojiCommon(id) ON DELETE CASCADE
);

##4) 창작마당 이모티콘 : CreatedEmoji 테이블
- 창작마당 이모티콘의 고유 정보를 관리

CREATE TABLE CreatedEmoji (
    id INT AUTO_INCREMENT PRIMARY KEY,
    emoji_common_id INT NOT NULL, -- EmojiCommon과 연동
    FOREIGN KEY (emoji_common_id) REFERENCES EmojiCommon(id) ON DELETE CASCADE
);

##5) 업로더 : Uploader 테이블
- 업로더 정보, 공통적으로 사용되므로 따로 테이블로써 분리하여 관리

CREATE TABLE Uploader (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL, -- 업로더 이름
    email VARCHAR(255), -- 업로더 이메일
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP -- 계정 생성일
);

##6) 카테고리 : EmojiCategories 테이블
- 이모티콘 카테고리를 관리

CREATE TABLE EmojiCategories (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL, -- 카테고리 이름
    description TEXT -- 설명
);

##7) 신고 : Report 테이블
- 신고 데이터를 관리

CREATE TABLE Report (
    id INT AUTO_INCREMENT PRIMARY KEY,
    emoji_id INT, -- 신고된 이모티콘 ID
    reporter_id INT NOT NULL, -- 신고한 사용자 ID
    message TEXT NOT NULL, -- 신고 내용
    status VARCHAR(50) DEFAULT 'pending', -- 처리 상태
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP, -- 신고 생성일
    FOREIGN KEY (emoji_id) REFERENCES EmojiCommon(id),
    FOREIGN KEY (reporter_id) REFERENCES Uploader(id)
);

##8) 짤방 공통 : ZzalCommon 테이블
- 짤방의 공통 속성을 관리

CREATE TABLE ZzalCommon (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL, -- 짤방 이름
    category_id INT, -- 카테고리 ID (ZzalCategories와 연동)
    tags VARCHAR(255), -- 태그
    uploader_id INT NOT NULL, -- 업로더 ID
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES ZzalCategories(id),
    FOREIGN KEY (uploader_id) REFERENCES Uploader(id)
);

##9) 짤방 카테고리 : ZzalCategories 테이블
- 짤방의 카테고리를 관리

CREATE TABLE ZzalCategories (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL, -- 카테고리 이름
    description TEXT -- 설명
);

#4. 테이블 관계

* EmojiCommon ↔ FreeEmoji
- 무료 이모티콘 데이터를 공통 속성과 연결.
- 관계: EmojiCommon.id = FreeEmoji.emoji_common_id

* EmojiCommon ↔ PaidEmoji
- 유료 이모티콘 데이터를 공통 속성과 연결.
- 관계: EmojiCommon.id = PaidEmoji.emoji_common_id

* EmojiCommon ↔ CreatedEmoji
- 창작마당 이모티콘 데이터를 공통 속성과 연결.
- 관계: EmojiCommon.id = CreatedEmoji.emoji_common_id

* EmojiCommon ↔ EmojiCategories
- 이모티콘과 카테고리를 연결.
- 관계: EmojiCommon.category_id = EmojiCategories.id

* Uploader ↔ EmojiCommon
- 업로더와 이모티콘을 연결.
- 관계: Uploader.id = EmojiCommon.uploader_id

* Report ↔ EmojiCommon
- 신고와 이모티콘을 연결.
- 관계: Report.emoji_id = EmojiCommon.id

* ZzalCommon ↔ ZzalCategories
- 짤방과 카테고리를 연결.
- 관계: ZzalCommon.category_id = ZzalCategories.id

* Uploader ↔ ZzalCommon
- 업로더와 짤방을 연결.
- 관계: Uploader.id = ZzalCommon.uploader_id



#5. 데이터화 과정

##1) 이모티콘/짤방 다운로드
* 무료 이모티콘 : 무료 배포된 사이트에서 다운로드
* 유료 이모티콘 : 이모티콘 판매하는 URL을 가져옴.
* 짤방 : 저장(캡쳐) 또는 URL 수집.

##2) JSON으로 정리
* 테이블에 삽입 전에 먼저 JSON으로 정리
예시)
[
    {
        "name": "웃는 고양이",
        "url": "https://example.com/emoji/happy_cat.png",
        "category": "동물",
        "tags": "고양이, 웃음, 귀여움",
        "uploader": "사용자1",
        "uploaded_at": "2025-05-07"
    },
    {
        "name": "화난 강아지",
        "url": "https://example.com/emoji/angry_dog.png",
        "category": "동물",
        "tags": "강아지, 분노, 귀여움",
        "uploader": "사용자2",
        "uploaded_at": "2025-05-07"
    }
]

##3) JSON ->MySQL 삽입

* 데이터삽입 예제 (스크립트X)

INSERT INTO EmojiBase (name, category, tags, uploader_id) 
VALUES ('웃는 고양이', '동물', '고양이, 웃음, 귀여움', 1);

INSERT INTO FreeEmoji (emoji_base_id, url) 
VALUES (1, 'https://example.com/free/happy_cat.png');

INSERT INTO PaidEmoji (emoji_base_id, purchase_url, price) 
VALUES (1, 'https://example.com/buy/happy_cat', 1.99);

* 수집한 데이터를 JSON으로 정리한 후, 파이썬이나 자바의 스크립트를 이용해 MySQL에 삽입








**python 예제 (JSON -> MySQL)


import pymysql
import json

- MySQL 연결
connection = pymysql.connect(
    host="localhost",
    user="root",
    password="password",
    database="emoji_db"
)
cursor = connection.cursor()

- JSON 파일 읽기
with open("emoji_data.json", "r", encoding="utf-8") as file:
    data = json.load(file)

- 데이터 삽입
for emoji in data:
    sql = """
        INSERT INTO Emoji (name, url, category, tags, uploader_id)
        VALUES (%s, %s, %s, %s, %s)
    """
    cursor.execute(sql, (
        emoji["name"], 
        emoji["url"], 
        emoji["category"], 
        emoji["tags"], 
        1  # uploader_id는 테스트용
    ))

- 커밋 및 종료
connection.commit()
cursor.close()
connection.close()


#6. API 설계

##1) Spring Boot를 사용해 데이터 접근 및 관리를 위한 엔드포인트를 만듦

* 엔드포인트 예제

- 전체 이모티콘 목록 가져오기
** GET /emojis

- 무료 이모티콘 목록 가져오기
** GET /free_emojis
** 필터: 카테고리, 태그, 업로더

- 유료 이모티콘 목록 가져오기
** GET /paid_emojis
** 필터: 카테고리, 태그, 업로더, 가격대

- 이모티콘 세부 정보 가져오기
** GET /emojis/{id}

- 새로운 이모티콘 추가
** POST  /emojis

- 신고 생성
** POST  /report

- 관리자 페이지
** GET  /admin
