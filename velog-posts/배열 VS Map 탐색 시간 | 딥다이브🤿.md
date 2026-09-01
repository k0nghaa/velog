<p><span style="font-size: 80%;"> 실제 코드 링크  <a href="https://github.com/hangyedan/assignment_test/pull/16">https://github.com/hangyedan/assignment_test/pull/16</a> </span></p>
<hr />
<p>그동안은 어떤 것(ex. 컴포넌트, 텍스트 등)을 나열해야할 때, 배열 안에 나열할 것들을 넣고 <code>.map</code> 매서드로 펼치는 방식을 주로 사용했다. </p>
<p>이 과제 테스트에서 특정 버튼을 눌렀을 때 그 버튼의 내용을 렌더링하는 <strong>탭</strong> 컴포넌트를 만들어야 했고, 마찬가지로 동일한 컴포넌트에 텍스트만 바뀌며 펼쳐야 한다는 생각에 데이터를 배열로 저장하고 <code>.map</code> 메서드를 사용했다.
배열 내 객체에 id를 추가하게 된 이유도 <code>.map</code> 때문이었다. 그리고 코드리뷰가 이렇게 달렸다.</p>
<blockquote>
<p>**&quot;각 컨텐츠가 id를 가지고 있어서 배열 대신 Map을 사용한다면 탐색 시간을 줄일 수 있을 것 같습니다.&quot; **</p>
</blockquote>
<hr />
<h2 id="map-이요">Map 이요?</h2>
<p>사실 Map(+ Set) 을 거의 사용하지 않았었다. 지피티를 이용해 코딩테스트 몇번 했을 때 내가 설계한 코드보다 더 나은 코드를 얘기할 때 봤던 정도. 
그렇게 잊고 있다가 이전 라이브리뷰 시간에 Map 에 대한 이야기가 나왔을 때 '아 Map 공부해야하는데' 생각했다. 그리고 코드리뷰에 까지 언급되어 이번 기회에 제대로 공부하고자 한다.</p>
<hr />
<h2 id="이번-상황에서-map-vs-배열">이번 상황에서 Map vs 배열</h2>
<p>현재 코드에서 SECTIONS 배열은 두가지 일을 하고 있었다.</p>
<pre><code>const SECTIONS = [
  {
    id: 1,
    buttonText: 'HTML',
    pText:
      'The HyperText Markup Language ...',
  },
  {
    id: 2,
    buttonText: 'CSS',
    pText:
      'Cascading Style Sheets is ...',
  },
  {
    id: 3,
    buttonText: 'JavaScript',
    pText:
      'JavaScript, often abbreviated as JS, is ...',
  },
];
</code></pre><ol>
<li><p>id로 특정 항목 찾기 ⇒ 활성 탭 내용 보여주기 ⇒ <strong>조회</strong></p>
<pre><code>const activeContent = SECTIONS.find(
(section) =&gt; section.id === activeSection,
);
...
   &lt;div&gt;
     &lt;p&gt;{activeContent?.pText}&lt;/p&gt;
   &lt;/div&gt;</code></pre></li>
<li><p>순서대로 렌더링하기 ⇒ 버튼 목록 펼치기 ⇒ <strong>순회</strong></p>
<pre><code>{SECTIONS.map((section) =&gt; (
&lt;button key={section.id}&gt;{section.buttonText}&lt;/button&gt;
))}</code></pre></li>
</ol>
<p>Map에는 .map()나 find() 같은 배열 메서드가 없어 렌더링할 때는 <code>[...map]</code> 처럼 배열로 변환해야한다. 그래서 배열(객체)의 key/value 를 모두 읽거나 보여줘야 하는 '순회'의 경우는 배열이 편하다.
그리고 특정 항목만 찾아야 하는 '조회'의 경우는 ** <em>O(1)*</em>인 Map의 get()을 사용하는 게 이론적으로 빠르다. </p>
<p>  *<span style="color: gray; font-size: 80%;">_ O(1): 특정 데이터를 찾을 때, 총 데이터 개수와 상관없이 항상 일정한 시간이 걸린다는 뜻_
<em>cf) O(n): 데이터 개수(n)에 비례해 시간이 걸린다는 뜻 
즉, n이 클수록 시간도 늘어남</em>
  </span></p>
<p>그렇기 때문에 모든 버튼 값(<code>buttonText</code>)을 읽어 보여줘야하는 렌더링 부분은 Map으로 변경하면 불편해진다.</p>
<pre><code>{/* 렌더링을 위해 일부러 배열로 변환해야 함 (map 메서드가 없기 때문에) */}
{[...SECTIONS_MAP].map(([id, section]) =&gt; (
          &lt;button key={id}&gt;{section.buttonText}&lt;/button&gt;</code></pre><p>그렇다면 활성 탭 내용(<code>pText</code>)은 Map으로 보여주는 게 효율적인가? 
사실상 이번 과제에는 SECTIONS 내 객체가 3개 뿐이기 때문에 Map을 사용해도 성능적으로 차이가 나지 않는다.Map은 수백, 수천개의 데이터를 다룰 때 성능 면으로 차이가 난다고 한다. </p>
<p>... 네?&gt;</p>
<p><strong>'그래서 그 수백, 수천개 데이터라는게 정확히 언제부터, 얼마나 차이가 나는거지?'</strong>
실제 그 차이를 눈으로 봐야 이해가 되는 나는 tinybench 라이브러리를 설치해 벤치마킹 검증을 해보았다. </p>
<hr />
<h2 id="탐색-로직-벤치마킹">탐색 로직 벤치마킹</h2>
<p>실제 SECTIONS 구조를 흉내낸 더미 데이터(<code>createSections</code>)를 생성해 배열과 Map 버전을 비교해봤다.</p>
<pre><code class="language-ts">import { Bench } from 'tinybench';

function createSections(size) {
  return Array.from({ length: size }, (_, i) =&gt; ({
    id: i + 1,
    buttonText: `Section${i + 1}`,
    pText: `Description text for section ${i + 1}`,
  }));
}

const SIZES = [10, 100, 1000, 10000, 100000];

for (const size of SIZES) {
  const arr = createSections(size);
  const map = new Map(arr.map((section) =&gt; [section.id, section]));
  const targetId = size; // worst case: 마지막 id

  const bench = new Bench({ time: 300, warmupTime: 100 });

  bench
    .add(`array.find (n=${size})`, () =&gt; {
      arr.find((section) =&gt; section.id === targetId);
    })
    .add(`map.get (n=${size})`, () =&gt; {
      map.get(targetId);
    });

  await bench.run();

  console.log(`\n=== 데이터 크기: ${size} ===`);
  console.table(bench.table());
}</code></pre>
<p>결과는 아래와 같았다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/db95de41-c0d5-473c-9179-48f83bc5ccb1/image.png" /></p>
<table>
<thead>
<tr>
<th>n</th>
<th>array.find (ns)</th>
<th>map.get (ns)</th>
<th>배수</th>
</tr>
</thead>
<tbody><tr>
<td>10</td>
<td>38.43</td>
<td>36.11</td>
<td>1.06x</td>
</tr>
<tr>
<td>100</td>
<td>115.93</td>
<td>36.08</td>
<td>3.2x</td>
</tr>
<tr>
<td>1,000</td>
<td>776.55</td>
<td>38.38</td>
<td>20.2x</td>
</tr>
<tr>
<td>10,000</td>
<td>7,353.7</td>
<td>36.73</td>
<td>200.2x</td>
</tr>
<tr>
<td>100,000</td>
<td>139,194</td>
<td>36.71</td>
<td>3,791.6x</td>
</tr>
</tbody></table>
<p>확실히 데이터 1000개 부터는 성능적으로 차이가 느껴질 것 같다.</p>
<p>.
.
.
<img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/d96bf9cf-19bb-4b0d-a710-5aca6ab4f366/image.png" /></p>
<p>으음 .. ns? 나노초 이게 실질적으로 화면으로 봤을 때 큰 차이가 날까??? </p>
<p>*<em>'알고리즘으로 돌렸을 때 차이가 나는 건 알겠는데 실제 컴포넌트 화면으로도 보고싶은데?'
*</em></p>
<hr />
<h2 id="실제-컴포넌트-렌더링-시간에-얼마나-영향을-주는가">실제 컴포넌트 렌더링 시간에 얼마나 영향을 주는가?</h2>
<p>실제 상황에 대입해 필요성을 확실히 느끼고 싶었다.</p>
<p>그래서 <strong>&quot;탭을 누르면 이 섹션과 연관된 콘텐츠 몇 만개를 확인해서 조회한 후 보여준다&quot;</strong> 와 같이 실제로 탭 컴포넌트에서 사용할만한 시나리오를 반영해 비교해보고자 했다.</p>
<h4 id="조건">조건</h4>
<p>1) 탭(버튼)은 그대로 3개로 유지해 렌더링 비용 결과 왜곡을 방지했다.
2) 하드코딩 상수 SECTIONS 가 아닌 sections prop을 주입하여 동적 데이터를 받을 수 있는 로직으로 변경했다.
3) &quot;탭 클릭 시&quot; 함께 실행되는 부가 조회 횟수(<code>RELATED_LOOKUP_COUNT</code>) 를 20,000 으로 동일하게 설정했다.</p>
<h3 id="tabstsx---배열-find-버전">Tabs.tsx - 배열 (<code>.find</code>) 버전</h3>
<pre><code class="language-ts">
const DEFAULT_SECTIONS= [
  {
    id: 1,
    buttonText: 'HTML',
    pText:
      '...',
  },
 ...
];

// 이 섹션과 연관된 인기 게시글 N개를 확인해서 개수를 세는 기능이 있다고 가정
const RELATED_LOOKUP_COUNT = 20_000;

interface TabsProps {
  sections?: Section[];
  benchmarkSections?: Section[];
}

export default function Tabs({
  sections = DEFAULT_SECTIONS,
  benchmarkSections,
}: TabsProps) {
  const [activeSection, setActiveSection] = useState(() =&gt; sections[0]?.id ?? 1);
  const [clickLatencyMs, setClickLatencyMs] = useState&lt;number | null&gt;(null);

  const dataForBenchmark = benchmarkSections ?? sections;

  const activeContent = sections.find((section) =&gt; section.id === activeSection);

  // ⭐ 실제 탭 클릭 시 실행되는 핸들러. setActiveSection 전에 &quot;관련 항목 조회&quot; 작업을 동기적으로 수행함.
  const handleTabClick = (id: number) =&gt; {
    const start = performance.now();

    let matchCount = 0;
    for (let i = 0; i &lt; RELATED_LOOKUP_COUNT; i++) {
      const randomId = Math.floor(Math.random() * dataForBenchmark.length) + 1;
      // 비교 대상: O(n) 선형 탐색
      if (dataForBenchmark.find((section) =&gt; section.id === randomId)) {
        matchCount++;
      }
    }

    const elapsed = performance.now() - start;
    setClickLatencyMs(elapsed);
    setActiveSection(id); 
  };

  return (
      &lt;div&gt;
        {sections.map((section) =&gt; (
          &lt;button
            key={section.id}
            onClick={() =&gt; handleTabClick(section.id)}
            style={activeSection === section.id ? { color: 'blue' } : {}}
          &gt;
            {section.buttonText}
          &lt;/button&gt;
        ))}
      &lt;/div&gt;
      &lt;div&gt;
        &lt;p&gt;{activeContent?.pText}&lt;/p&gt;
      &lt;/div&gt;
  );
}</code></pre>
<h3 id="tabswithmaptsx---mapget-버전">TabsWithMap.tsx - <code>Map.get</code> 버전</h3>
<pre><code class="language-ts">... (Tabs과 동일)

 // sections 참조가 바뀔 때만 Map을 새로 만듦 (렌더링마다 재생성 방지)
  const sectionsMap = useMemo(
    () =&gt; new Map(sections.map((section) =&gt; [section.id, section])),
    [sections],
  );

  // 클릭 시 조회할 대규모 Map도 마찬가지로 참조가 바뀔 때만 재생성
  const benchmarkMap = useMemo(
    () =&gt; new Map(dataForBenchmark.map((section) =&gt; [section.id, section])),
    [dataForBenchmark],
  );

  const activeContent = sectionsMap.get(activeSection);

  // ⭐ 조회 방식: Map.get
  const handleTabClick = (id: number) =&gt; {
    const start = performance.now();

    let matchCount = 0;
    for (let i = 0; i &lt; RELATED_LOOKUP_COUNT; i++) {
      const randomId = Math.floor(Math.random() * dataForBenchmark.length) + 1;
      // 비교 대상: O(1) 해시 조회
      if (benchmarkMap.get(randomId)) {
        matchCount++;
      }
    }

    const elapsed = performance.now() - start;
    setClickLatencyMs(elapsed);
    setActiveSection(id);
  };

  ... (Tabs과 동일)</code></pre>
<h4 id="참고-generatesections-app">참고 <code>generateSections</code>, <code>App</code></h4>
<pre><code class="language-ts">function generateSections(size: number): Section[] {
  return Array.from({ length: size }, (_, i) =&gt; ({
    id: i + 1,
    buttonText: `Section${i + 1}`,
    pText: `Description text for section ${i + 1}`,
  }));
}

export default function App() {
  // 한 번만 생성 -&gt; 리렌더링 시 재생성되어 측정값이 왜곡되지 않도록
  const [bigDataset] = useState(() =&gt; generateSections(1_000_000));

  return (
    &lt;div&gt;
      &lt;Tabs benchmarkSections={bigDataset} /&gt;
      &lt;hr /&gt;
      &lt;TabsWithMap benchmarkSections={bigDataset} /&gt;
    &lt;/div&gt;
  );
}

</code></pre>
<h3 id="결과">결과</h3>
<p><img alt="" src="https://velog.velcdn.com/images/k0nghaa/post/8b34cc39-0e19-4b26-be8f-bc3660990a0a/image.gif" /></p>
<p>이렇게 보니 확실히 와닿았다!</p>
<hr />
<h2 id="그렇다면-언제-map-을-써야할까">그렇다면 언제 Map 을 써야할까?</h2>
<p>이제 이런 게 궁금했다. 
'그렇다면 처음부터 늘어날 것을 대비해 Map을 써야하는 걸까? 언제 Map을 써야하는거지?'</p>
<p>실무에서 &quot;일단 배열/객체로 짜고 필요할 때 바꾼다&quot;는 원칙을 따른다고 한다.</p>
<ul>
<li>프론트엔드에서 다루는 리스트(할 일 목록, 탭, 댓글 등)는 수십~수백 개 수준이라 차이가 없거나, </li>
<li>배열 보다 Map이 다루기 어렵기도 하고,</li>
<li>미리 최적화했다가 최적화가 필요없을 정도로만 남게 되면 복잡한 코드만 남기 때문이다.</li>
</ul>
<p>실제 Map으로 바꿔야겠다고 판단하는 신호가 있다면 아래와 같다고 한다.</p>
<blockquote>
<ul>
<li><code>find()</code>나 <code>filter()</code> 가 렌더링 시간의 상당 부분을 차지하는 게 Profiler나 크롬 Performance 탭에 찍힐 때</li>
<li>데이터 규모가 실측으로 커질 때 (ex. 단순히 즐겨찾기 몇 개가 아니라 전체 상품 목록 수만 개를 다뤄야 할때)</li>
<li>한 번 조회하고 끝이 아니라 매 렌더링, 매 타이핑, 매 스크롤마다 특정 항목을 찾아야 할 때</li>
<li>실시간 채팅, 웹소켓 연결처럼 항목이 초 단위로 추가 혹은 제거되는 경우</li>
<li>타이핑이 버벅이거나, 클릭 후 반응이 눈에 띄게 늦어질 때</li>
</ul>
</blockquote>
<p>물론 위 상황들도 바로 Map을 사용하는 게 아니라 다른 방안이 있는지 트레이드 오프를 확인해야 할 것이다. 
다만 'ID로 항목을 찾는 로직이 반복적으로 실행되고 있는가?', '그 작업이 비효율적인 방법으로 필요 이상 오래 걸리고 있는가?' 를 따져봤을 때, 그 작업량 자체를 줄일 수 있다면 Map을 사용해 알고리즘 개선을 검토해야겠다고 생각했다.</p>
<hr />
<h2 id="마무리"><del>마무리</del></h2>
<p>이번 과제테스트에서는 탭이 3개고 그에 맞는 단순 텍스트를 보여주는 것이었기 때문에 가독성 좋은 배열(.find)를 사용해도 괜찮았다. 
그러나 몰라서 그냥 배열을 쓰는 것과 트레이드 오프로 비교하고 배열을 쓰는 건 다르다.
코드를 짤 땐 항상 지금의 상황을 인지하며 구현하고, 앞으로의 상황도 생각해 대비하는 게 좋겠다고 느꼈다!</p>