---
layout: default
title: "태그별 모아보기"
---

<div style="background: #ffffff; padding: 24px; border-radius: 12px; box-shadow: 0 1px 3px rgba(0,0,0,0.05);">
  <h2 style="font-size: 1.1rem; margin-top: 0; margin-bottom: 16px; color: #1e293b;">🏷️ 태그별 기록 모아보기</h2>
  
  <!-- 선택된 태그의 글들을 월별로 모아서 보여주는 영역 -->
  <div id="tag-posts-container">
    <p id="tag-select-msg" style="color: #64748b; font-size: 0.9rem; text-align: center; padding: 20px 0;">
      사이드바의 태그를 선택하여 기록을 월별로 모아보세요.
    </p>
    
    <div id="posts-list-wrapper" style="display: none;">
      <h3 id="selected-tag-title" style="font-size: 1rem; color: #0284c7; margin-bottom: 20px; font-weight: 600;"></h3>
      
      <!-- 월별 그룹화 리스트가 들어갈 자리 -->
      <div id="monthly-groups"></div>
    </div>
  </div>
</div>

<!-- 사이트의 모든 포스트 데이터를 자바스크립트로 전달 -->
<script>
    const allPosts = [
        {% for post in site.posts %}
        {
            title: "{{ post.title }}",
            date: "{{ post.date | date: '%Y-%m-%d %H:%M' }}",
            yearMonth: "{{ post.date | date: '%Y년 %m월' }}",
            url: "{{ post.url }}",
            tags: [{% if post.tags %}{% for t in post.tags %}"{{ t | strip }}"{% unless forloop.last %},{% endunless %}{% endfor %}{% endif %}]
        }{% unless forloop.last %},{% endunless %}
        {% endfor %}
    ];

    document.addEventListener("DOMContentLoaded", function() {
        const urlParams = new URLSearchParams(window.location.search);
        const currentTag = urlParams.get('tag');

        if (currentTag) {
            // 해당 태그가 포함된 포스트 필터링
            const filteredPosts = allPosts.filter(post => post.tags.includes(currentTag));

            document.getElementById('tag-select-msg').style.display = 'none';
            document.getElementById('posts-list-wrapper').style.display = 'block';
            document.getElementById('selected-tag-title').innerText = `#${currentTag} 태그가 포함된 기록 (${filteredPosts.length}개)`;

            const monthlyGroupsEl = document.getElementById('monthly-groups');
            
            if (filteredPosts.length === 0) {
                monthlyGroupsEl.innerHTML = '<p style="color: #64748b; text-align: center; font-size: 0.9rem;">이 태그가 작성된 기록이 없습니다.</p>';
                return;
            }

            // 월별로 그룹화 (YYYY년 MM월 기준)
            const grouped = {};
            filteredPosts.forEach(post => {
                if (!grouped[post.yearMonth]) {
                    grouped[post.yearMonth] = [];
                }
                grouped[post.yearMonth].push(post);
            });

            // HTML 생성
            let html = '';
            for (const [ym, posts] of Object.entries(grouped)) {
                html += `<div style="margin-bottom: 20px;">`;
                html += `<h4 style="font-size: 0.9rem; color: #334155; border-bottom: 1px solid #e2e8f0; padding-bottom: 4px; margin-bottom: 10px;">📅 ${ym}</h4>`;
                html += `<ul style="list-style: none; padding: 0; margin: 0;">`;
                
                posts.forEach(post => {
                    const mmdd = post.date.substring(5, 10); 
                    html += `<li style="margin-bottom: 8px; padding: 8px 12px; background: #f8fafc; border-radius: 6px; display: flex; justify-content: space-between; align-items: center;">`;
                    html += `<a href="/?date=${mmdd}" style="color: #1e293b; text-decoration: none; font-weight: 500; font-size: 0.88rem;">${post.date}</a>`;
                    html += `<a href="/?date=${mmdd}" style="font-size: 0.8rem; color: #3b82f6; text-decoration: none;">해당 날짜 보기 ➔</a>`;
                    html += `</li>`;
                });

                html += `</ul></div>`;
            }
            monthlyGroupsEl.innerHTML = html;
        }
    });
</script>
