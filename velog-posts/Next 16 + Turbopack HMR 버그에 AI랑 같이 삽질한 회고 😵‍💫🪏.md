<p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/2042e502-58de-4951-886e-885abbd219ac/image.png" /></p>
<blockquote>
<p>Next 16 + Turbopack + React Compiler 를 쓰는데 이 에러가 난다면 ...
Next 버전 업데이트를 해보세요</p>
</blockquote>
<p>어제부터 골머리를 앓고 있던 에러가 팀원 한 마디로 약 10분 만에 해결이 됐다.
허탈감과 현타가 왔지만 이런 기분만 느끼고 끝내면 무조건 반복할 것이라 판단해•• 
이번에 <strong>디버깅 방식 자체를 돌아보는 회고를</strong> 남겨본다.</p>
<hr />
<h2 id="프로젝트-상황">프로젝트 상황</h2>
<p>간단히 프로젝트 상황부터 설명하자면</p>
<p>MVP 2차부터 참여하기로 한 사이드 프로젝트로, 
프로젝트 세팅 및 MVP 1차에 대한 코드가 작성되어 있는 상태였다. 
Next.js 16, React 19, Node.js 24, Turbopack(dev 번들러), pnpm 등을 사용하고 있었다.
<img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/fb9e4af7-2b14-43b0-a55c-03bac2e27954/image.png" /></p>
<h2 id="문제-상황">문제 상황</h2>
<p>pnpm와 React Compiler를 처음 사용해보는 상황이었다.</p>
<p>어찌저찌 패키지 설치 후 회원가입부터 해보려는데 
이메일 인풋에 유효한 값을 넣어도 버튼이 활성화되지 않았다.
<img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/49bd0f06-2720-4ffb-a7c4-5973fbb549b2/image.png" />
_<span style="color: gray;">팀원분이 올려주신 이슈 재현 영상 캡쳐</span> _</p>
<p>관련 코드 전반을 클루드에게 보여주고 원인을 물으니 <code>emailDisabled</code> 조건과 react-hook-form 사용 시 defaultValues가 없어 <code>watch('email')</code> 초기값이 undefined가 되기 때문으로 파악했다.</p>
<pre><code class="language-ts">// emailDisabled 조건
  const emailDisabled =
    !watchedEmail || !!errors.email || (isCodeSent &amp;&amp; !canResend) || isVerified;</code></pre>
<p>defaultValues를 추가해도 작동이 안돼 errors.email 을 콘솔로 찍어보았다.
처음엔 콘솔만 추가했는데도 버튼이 활성화됐고, 콘솔에는 undefined가 한 2초 간격으로 계속 떴다. 클루드는 이를 React Compiler 때문이라 판단했다. 
<img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/50c4f079-931c-4c4b-86ce-e2ad7b074d18/image.png" /></p>
<ol>
<li>컴포넌트 내부에 있는 'use no memo' 지시어를 파일 최상단으로 올렸다.
 -&gt; 버튼 비활성화 당연히 해결 안되고 콘솔도 찍히지 않았다.</li>
<li><code>watch</code> 함수 그대로 전달하는 대신 값을 직접 구독해서 내렸다.
 -&gt; 여전히 해결 X 콘솔도 찍히지 않음</li>
</ol>
<p>이후 props 수정, 타입 에러 수정, <code>'use no memo'</code>와 <code>'use client'</code> 순서 변경 등을 반복했지만 해결이 안 됐다. 디버깅 비용이 너무 커지자 React Compiler를 끄는 걸 추천했고 이때부터 진짜 문제가 시작됐다.</p>
<p><code>reactCompiler</code>를 껐다. 그리고 이 에러를 만났다 !!</p>
<pre><code>Module [project]/node_modules/.pnpm/@tanstack+query-core@5.91.2/...
was instantiated because it was required from module
[project]/components/_layout/queryProvider.tsx [app-client] (ecmascript),
but the module factory is not available. It might have been deleted in an HMR update.</code></pre><p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/b336ffa6-d659-4dd8-9d52-7925ce585199/image.png" /></p>
<h2 id="에러를-대하는-기존-태도">에러를 대하는 기존 태도</h2>
<p>그동안 에러를 만났을 때 해결하는 방식은 두 가지 중 하나였다.</p>
<blockquote>
<p>1) 콘솔 에러 메시지를 복붙해서 AI한테 물어보기
2)  인터넷에 에러 메시지 검색해보기</p>
</blockquote>
<p>대부분은 1로 해결이 됐고, 해결하면 왜 그랬는지 원인을 찾지 않고 넘어갔다.</p>
<p>이번에도 당연히(?) AI한테 물어봤다. AI는 &quot;React Compiler + HMR 충돌로 모듈 캐시가 깨진 상태&quot;로 판단했고, 패키지 삭제·재설치, 설정 변경 등을 반복했다.</p>
<pre><code class="language-bash">rm -rf .next
rm -rf node_modules
pnpm store prune
pnpm install
pnpm dev
# 무한루프 ...</code></pre>
<h2 id="해결">해결</h2>
<p>팀원들에게 잘 작동되는지 물어봤더니 다른 분들은 문제없다고 해서 내 환경 문제라고 생각했다. 다음날 다른 분이 올린 react compiler 관련 PR을 머지해보고 최신 상태로 돌려보기로 했다.</p>
<p>결국 한 팀원분도 동일한 에러가 뜬다는 걸 확인했고, <code>pnpm dev --webpack</code> 커맨드로 실행한 뒤 정확한 원인을 찾아보겠다 했다. 그리고 정확히 13분 후 해결했다는 댓글이 달렸다.</p>
<p>원인은 <strong>Next.js 16 + Turbopack의 버그</strong>였다.
Next 버전을 올려보니 에러가 사라졌고, Next.js 16.2 업데이트 내역에서 Turbopack 관련 200개 이상의 버그 픽스 항목을 확인할 수 있었다.
<strong>AI는 끝까지 &quot;React Compiler + HMR 모듈 캐시 충돌&quot; 로 판단했지만, 실제 원인은 단순한 패키지 버그였다.</strong></p>
<p><a href="https://nextjs.org/blog/next-16-2-turbopack#performance-improvements-and-bug-fixes">관련 내용 링크</a></p>
<ul>
<li>1차 충격 : <code>pnpm dev --webpack</code> 커맨드로 실행</li>
<li>2차 충격 : 13분 만에 해결 답변 ..</li>
<li>3차 충격 : 업데이트 내역 확인</li>
</ul>
<p>아무튼 나도 Next 업데이트하니 진짜 되더라 <del>~ ~</del>아임 피네~ㅋㅋ~~
<img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/684d0801-31f7-462d-960e-984830c08b5c/image.png" /></p>
<h2 id="문제점">문제점</h2>
<p>머 누굴 탓할거야
결국은 내 문제다 !! ! </p>
<h3 id="개인적인-태도-문제">개인적인 태도 문제</h3>
<ul>
<li>무작정 AI에 의존하는 버릇이 있다.</li>
<li>에러를 해결하면 원인을 찾지 않고 넘어가는 태도도 문제였다.</li>
</ul>
<h3 id="협업-관련-태도">협업 관련 태도</h3>
<p>정확하게 에러 이슈를 전달하지 못해 해결 시간이 지체됐다. 
안된다는 상황만 공유했지, 어떤 에러인지 정확히 공유하지 않았다.</p>
<h2 id="앞으로-바꿔야할-태도---디버깅-순서">앞으로 바꿔야할 태도 - 디버깅 순서</h2>
<h3 id="에러-메시지-정확히-읽어보기">에러 메시지 정확히 읽어보기</h3>
<p>복붙 금지. 당연하지만 잘 안 읽어보기 일쑤였다.</p>
<h3 id="내-코드-문제인지-확인하기">내 코드 문제인지 확인하기</h3>
<p>최근 수정한 부분에서 난 건지, 특정 컴포넌트에서만 발생하는지 먼저 확인한다.</p>
<h3 id="환경-문제인지-체크하기">환경 문제인지 체크하기</h3>
<p>나만 안 되는 건지, 다른 팀원들은 되는지 확인한다.</p>
<h3 id="번들러빌드-문제인지-체크하기">번들러/빌드 문제인지 체크하기</h3>
<p><code>module factory</code>, <code>runtime</code>, <code>HMR</code>, <code>compiler-runtime</code> 같은 키워드가 보이면 번들러/빌드 문제를 의심한다! 이건 코드 문제가 아님</p>
<h3 id="버전이슈-검색해보기">버전/이슈 검색해보기</h3>
<p>이도저도 안되면 GitHub 이슈나 공식 블로그를에서 관련 버그가 있는지, 버전 업데이트로 해결된 문제인지 확인한다.</p>
<hr />
<p>아무튼.ᐟ 큰 깨달음을 얻은 이슈였다. 
에러를 해결하는 것보다 어떻게 접근하는가가 더 중요하다는 걸 다시 한번 느꼈다.</p>
<p>그롬 .. 공부하러 가볼게요
*<em><del>회고 끗</del> *</em>
<img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/9ddee60a-469a-4a67-96bb-9e04e3d2ff5d/image.png" /></p>