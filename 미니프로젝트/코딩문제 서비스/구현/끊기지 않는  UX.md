

- zustand persist를 사용한 로그인상태를 로컬스토리지에 저장해서 로그인유지 구현



## 로딩 상태에서의  피드백 제공

- 커뮤니티 리스트: useQuery의 isPending일 때 CommunityPageSkeleton을 바로 보여줘서, 목록이 늦게 와도 레이아웃이 유지되고 “로딩 중임”이 시각적으로 전달됩니다 

- 커뮤니티 상세: isLoading일 때 CommunityDetailSkeleton을 사용해서 상세 페이지 레이아웃 자체가 유지됩니다 (CommunityDetail.tsx).

- 마이페이지 통계: isLoading일 때 전체 통계 레이아웃에 맞춘 UserStatisticsSkeleton을 사용해, 길게 걸리는 요청에서도 화면이 비지 않습니다 (UserStatistics.tsx).

- 퀴즈: 버튼에 isLoading/isGrading을 연결해서 Spinner로 상태를 보여주며 동시에 disabled 처리하여 중복 클릭/중복 요청을 막습니다 (Button.tsx, QuizScreen.tsx).

## 에러처리

- 퀴즈 영역에 ErrorBoundary를 감싸고, fallback에서 “다시 시도” 버튼으로 재요청을 할 수 있게 구성해 두어, 예외가 나도 전체 페이지가 깨지지 않습니다 (QuizScreen.tsx, ErrorBoundary.tsx).

- 커뮤니티 목록·상세, 유저 통계 등은 ErrorComp를 통해 페이지 이름 + 메시지 + “새로고침 / 메인으로 이동” 버튼을 제공해, 단순 에러 문구가 아니라 복귀 동선을 안내합니다 (ErrorComp.tsx).


## 토스트를 사용한 결과를 즉시 전달

- useToastStore + ToastContainer + AnimatePresence로 전역 토스트 시스템이 구성되어 있고, 로그인 필요, 게시글/댓글 등록/삭제, 오답 등록/삭제, 채점 에러 등 대부분의 유저 액션에 대해 피드백이 있습니다.

- 예: 보호 라우트(ProtectedRoute)에서 미로그인 시 토스트로 경고 후 /main으로 리다이렉트, 커뮤니티 CRUD 및 오답 관리에서 성공/실패 토스트 제공.


## 그외


- 메인 랜딩은 스크롤 스냅 + 애니메이션 + 섹션별 소개로, 컨셉 전달과 이동 버튼(이동하기)이 자연스럽게 이어집니다 

- 커뮤니티 검색은 useDebounce로 디바운싱되어, 입력할 때마다 API가 튀지 않아 체감 성능이 좋습니다.

- useInfiniteScroll 훅까지 준비되어 있어서, 무한 스크롤이 필요한 리스트에서 끊김을 줄이도록 설계되어 있습니다 (useInfiniteScroll.ts).