---
layout: default
title: "태그별 기록 모아보기"
---

<div style="background: #ffffff; padding: 24px; border-radius: 12px; box-shadow: 0 1px 3px rgba(0,0,0,0.05);">
  <h2 id="page-title" style="font-size: 1.1rem; margin-top: 0; margin-bottom: 16px; color: #1e293b;">🏷️ 태그별 기록 모아보기</h2>
  
  <!-- 태그를 선택하지 않았을 때 안내 메시지 -->
  <p id="tag-select-msg" style="color: #64748b; font-size: 0.9rem; text-align: center; padding: 20px 0;">
    사이드바의 태그를 선택하여 기록을 모아보세요.
  </p>
  
  <div id="posts-list-wrapper" style="display: none;">
    <!-- 연도 이동 내비게이션 (삼각형 키) -->
    <div style="display: flex; align-items: center; justify-content: space-between; background: #f8fafc; padding: 12px 16px; border-radius: 8px; margin-bottom: 24px; border: 1px solid #e2e8f0;">
      <a href="#" id="prev-year-btn" style="text-decoration: none; color: #0284c7; font-weight: 700; font-size: 0.95rem; padding: 4px 8px;">◀ 이전 연도</a>
      <h3 id="selected-year-title" style="font-size: 1rem; color: #1e293b; margin: 0; font-weight: 600;"></h3>
      <a href="#" id="next-year-btn" style="text-decoration: none; color: #0284c7; font-weight: 700; font-size: 0.95rem; padding: 4px 8px;">다음 연도 ▶</a>
    </div>

    <!-- 선택된 연도의 포스트들이 본문 그대로 렌더링될 영역 -->
    <div id="posts-container"></div>
  </div>
</div>

<!-- 사이트의 모든 포스트 데이터를 자바스크립트로 전달 -->
<script>
    const allPosts = [
        {% for post in site.posts %}
        {
            date: "{{ post.date | date: '%Y-%m-%d %H:%M' }}",
            year: "{{ post.date | date: '%Y' }}",
            rawDate: "{{ post.date | date: '%Y-%m-%d %H:%M' }}",
            content: {{ post.content | jsonify }},
            tags: [{% if post.tags %}{% for t in post.tags %}"{{ t | strip }}"{% unless forloop.last %},{% endunless %}{% endfor %}{% endif %}]
        }{% unless forloop.last %},{% endunless %}
        {% endfor %}
    ];

    document.addEventListener("DOMContentLoaded", function() {
        const urlParams = new URLSearchParams(window.location.search);
        const currentTag = urlParams.get('tag');

        if (!currentTag) return;

        // 해당 태그가 포함된 포스트 필터링
        const filteredPosts = allPosts.filter(post => post.tags.includes(currentTag));

        document.getElementById('tag-select-msg').style.display = 'none';
        document.getElementById('posts-list-wrapper').style.display = 'block';
        document.getElementById('page-title').innerText = `#${currentTag} 태그 기록`;

        if (filteredPosts.length === 0) {
            document.getElementById('posts-container').innerHTML = '<p style="color: #64748b; text-align: center; font-size: 0.9rem; padding: 20px;">이 태그가 작성된 기록이 없습니다.</p>';
            document.getElementById('prev-year-btn').style.display = 'none';
            document.getElementById('next-year-btn').style.display = 'none';
            return;
        }

        // 존재하는 연도 목록 추출 (오름차순 정렬)
        const availableYears = [...new Set(filteredPosts.map(p => p.year))].sort();
        
        // URL에서 연도 가져오기, 없으면 가장 최근 연도 선택
        let selectedYear = urlParams.get('year');
        if (!selectedYear || !availableYears.includes(selectedYear)) {
            selectedYear = availableYears[availableYears.length - 1]; // 가장 최신 연도
        }

        document.getElementById('selected-year-title').innerText = `${selectedYear}년 기록`;

        // 연도 이동 버튼 설정 (삼각형 키)
        const currentYearIndex = availableYears.indexOf(selectedYear);
        const prevYearBtn = document.getElementById('prev-year-btn');
        const nextYearBtn = document.getElementById('next-year-btn');

        if (currentYearIndex > 0) {
            prevYearBtn.href = `{{ site.baseurl }}/tags?tag=${currentTag}&year=${availableYears[currentYearIndex - 1]}`;
            prevYearBtn.style.visibility = 'visible';
            prevYearBtn.innerText = `◀ ${availableYears[currentYearIndex - 1]}년`;
        } else {
            prevYearBtn.style.visibility = 'hidden';
        }

        if (currentYearIndex < availableYears.length - 1) {
            nextYearBtn.href = `{{ site.baseurl }}/tags?tag=${currentTag}&year=${availableYears[currentYearIndex + 1]}`;
            nextYearBtn.style.visibility = 'visible';
            nextYearBtn.innerText = `${availableYears[currentYearIndex + 1]}년 ▶`;
        } else {
            nextYearBtn.style.visibility = 'hidden';
        }

        // 해당 연도의 포스트들만 필터링
        const yearPosts = filteredPosts.filter(p => p.year === selectedYear);
        const container = document.getElementById('posts-container');
        
        let html = '';
        yearPosts.forEach(post => {
            // 날짜 포맷팅 (YYYY. MM. DD. (요일) HH:MM)
            const dateObj = new Date(post.rawDate.replace(' ', 'T') + ':00');
            const y = dateObj.getFullYear();
            const m = String(dateObj.getMonth() + 1).padStart(2, '0');
            const d = String(dateObj.getDate()).padStart(2, '0');
            const weekdays = ['일', '월', '화', '수', '목', '금', '토'];
            const wDay = weekdays[dateObj.getDay()];
            const timePart = post.rawDate.split(' ')[1];
            const formattedDate = `${y}. ${m}. ${d}. (${wDay}) ${timePart}`;

            // default 화면에서 쓰던 다이어리 포스트 박스 스타일 적용
            html += `<article style="background: #ffffff; padding: 24px; border-radius: 12px; box-shadow: 0 1px 3px rgba(0,0,0,0.05); margin-bottom: 20px;">`;
            html += `<div style="font-size: 0.85rem; color: #64748b; margin-bottom: 12px; font-weight: 500;">${formattedDate}</div>`;
            html += `<div style="line-height: 1.7; font-size: 0.88rem;">${post.content}</div>`;
            html += `</article>`;
        });

        container.innerHTML = html;
    });
</script>
