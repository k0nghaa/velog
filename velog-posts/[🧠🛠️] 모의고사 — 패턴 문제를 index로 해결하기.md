<p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/bbef91d0-9880-4649-a86b-cac99fe822b3/image.png" /></p>
<h2 id="✍️-들어가며">✍️ 들어가며</h2>
<p>코딩테스트를 풀 때 나는 보통 “이렇게 해결하면 되지 않을까?” 같은 방식으로 접근하는 편이다.  </p>
<p>문제 구조를 완전히 이해하기보다는
일단 돌아가는 방법을 먼저 떠올리는 쪽에 가까웠다. </p>
<p>이제부터는 단순히 문제를 푸는 게 아니라
내 사고 과정을 정리하면서 <strong>개발자식 사고로 전환해보고자</strong> 이렇게 글로 작성하려고 한다.</p>
<hr />
<h2 id="🔗-문제">🔗 문제</h2>
<p>[프로그래머스 - 모의고사] 
<a href="https://school.programmers.co.kr/learn/courses/30/lessons/42840">https://school.programmers.co.kr/learn/courses/30/lessons/42840</a></p>
<hr />
<h2 id="💭-처음-접근했던-방식">💭 처음 접근했던 방식</h2>
<p>처음에는 각 수포자의 <strong>방식 배열</strong>과 answers를 비교해서 일치하는 값만 걸러내고 그 개수로 점수를 계산하려고 했다.  </p>
<p>틀린 경우는 제외해야 한다고 생각해서 <code>!==</code> 와 <code>continue</code>를 사용하려는 흐름으로 접근했다. </p>
<hr />
<h2 id="🤔-막혔던-이유">🤔 막혔던 이유</h2>
<p>이 방식에는 몇 가지 문제가 있었다.</p>
<h3 id="1-패턴-반복을-활용하지-못함">1) 패턴 반복을 활용하지 못함</h3>
<p>수포자의 답안은 고정된 배열이 아니라 반복되는 구조인데 단순히 배열을 비교하는 방식으로 접근했다.</p>
<h3 id="2-불필요한-배열-생성">2) 불필요한 배열 생성</h3>
<p>filter로 배열을 만든 뒤 length를 구하는 방식으로 굳이 필요 없는 과정을 추가하려 했다.</p>
<h3 id="3-continue-사용으로-흐름이-복잡해짐">3) continue 사용으로 흐름이 복잡해짐</h3>
<p>for문 내 if절에서 틀린 경우를 제외하기 위해 continue를 사용했지만 사실 조건문 안에서 충분히 처리할 수 있는 부분이었다.</p>
<pre><code>if (answers[i] !== p1[i % p1.length]) continue;</code></pre><hr />
<h2 id="💡-사고가-바뀐-포인트">💡 사고가 바뀐 포인트</h2>
<p>이 문제의 핵심은 &quot;패턴 반복&quot;이었다.
이 부분을 짚으면서 <strong>index 기반 접근이 필요하다</strong>는 것을 알게 됐다.</p>
<p>특히 아래 코드가 핵심이었다</p>
<pre><code class="language-js">answers[i] === p1[i % p1.length]</code></pre>
<p><code>i % length</code>를 사용하면 배열의 끝까지 갔을 때 다시 처음으로 돌아오면서 패턴을 자연스럽게 반복할 수 있다.</p>
<p>그리고 배열을 따로 만들지 않고, 맞을 때마다 count를 증가시키는 방식으로 바꾸면서 전체 로직이 훨씬 단순해졌다.</p>
<hr />
<h2 id="✅-최종-코드">✅ 최종 코드</h2>
<pre><code class="language-js">function solution(answers) {
    const p1 = [1,2,3,4,5];
    const p2 = [2,1,2,3,2,4,2,5];
    const p3 = [3,3,1,1,2,2,4,4,5,5];

    let score = [0,0,0];

    for (let i = 0; i &lt; answers.length; i++) {
      if (answers[i] === p1[i % p1.length]) score[0]++;
      if (answers[i] === p2[i % p2.length]) score[1]++;
      if (answers[i] === p3[i % p3.length]) score[2]++;
    }

    let max_score = Math.max(...score)
    let winners = score.map((s, i) =&gt; s === max_score ? i+1 : null)
    let answer = winners.filter(v =&gt; v !== null)

    return answer;
}</code></pre>
<hr />
<h2 id="🔍-다른-풀이를-보며">🔍 다른 풀이를 보며</h2>
<p>굳이 map + filter를 쓰지 않고도 더 직관적으로 처리할 수 있었다.</p>
<pre><code class="language-js">var max = Math.max(a1c, a2c, a3c);

if (a1c === max) answer.push(1);
if (a2c === max) answer.push(2);
if (a3c === max) answer.push(3);

return answer;</code></pre>
<p>ㄴ 불필요한 배열을 만들지 않고 바로 결과를 만드는 방식이라 더 간결하게 느껴졌다.</p>
<hr />
<h2 id="🧠-코테-풀-때-체크할-것">🧠 코테 풀 때 체크할 것</h2>
<h3 id="---굳이-저장해야-하나">[ ]  굳이 저장해야 하나?</h3>
<p>   → 배열 대신 count로 끝낼 수 있는지</p>
<h3 id="---continue가-필요한가">[ ]  continue가 필요한가?</h3>
<p>   → 조건문 안에서 해결 가능한지</p>
<h3 id="--index-기반-문제인가">[ ] index 기반 문제인가?</h3>
<p>   → 반복 패턴이 보이면 <code>%</code> 떠올리기</p>
<hr />
<h2 id="🧾-한-줄-회고">🧾 한 줄 회고</h2>
<p>** 불필요한 로직을 덜어내는 과정을 인식하게 되었다. **</p>