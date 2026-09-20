---
layout: default
title: "On This Day"
---

{% assign sorted_posts = site.posts | sort: "date" | reverse %}

<div id="posts-container">
  {% for post in sorted_posts %}
    {% assign post_year = post.date | date: "%Y" %}
    {% assign post_month = post.date | date: "%m" %}
    {% assign post_day = post.date | date: "%d" %}
    
    <article class="diary-entry" 
             data-year="{{ post_year }}" 
             data-month-day="{{ post_month }}-{{ post_day }}" 
             style="display: none; padding: 24px; border-radius: 12px; box-shadow: 0 1px 3px rgba(0,0,0,0.05); margin-bottom: 20px;">
      <div class="entry-meta" data-raw-date="{{ post.date | date: '%Y-%m-%d %H:%M' }}" style="font-size: 0.85rem; color: #64748b; margin-bottom: 12px; font-weight: 500;">
        {{ post.date | date: "%Y. %m. %d. %H:%M" }}
      </div>
      <div style="line-height: 1.7; font-size: 0.88rem;">
        {{ post.content | markdownify }}
      </div>
    </article>
  {% endfor %}

  <div id="no-posts-msg" style="display: none; text-align: center; padding: 40px; background: #fff; border-radius: 12px; color: #64748b; box-shadow: 0 1px 3px rgba(0,0,0,0.05);">
    <p>이 날짜의 기록이 없습니다.</p>
    <p style="font-size: 0.9rem; margin-top: 5px; color: #94a3b8;">첫 이야기를 남겨보세요!</p>
  </div>
</div>

<script>
    document.addEventListener("DOMContentLoaded", function() {
        const currentRealYear = new Date().getFullYear().toString(); // "2026"

        const metaElements = document.querySelectorAll('.entry-meta');
        metaElements.forEach(el => {
            const rawDateStr = el.getAttribute('data-raw-date');
            if (rawDateStr) {
                const dateObj = new Date(rawDateStr.replace(' ', 'T') + ':00');
                if (!isNaN(dateObj)) {
                    const y = dateObj.getFullYear();
                    const m = String(dateObj.getMonth() + 1).padStart(2, '0');
                    const d = String(dateObj.getDate()).padStart(2, '0');
                    
                    const weekdays = ['일', '월', '화', '수', '목', '금', '토'];
                    const wDay = weekdays[dateObj.getDay()];
                    const timePart = rawDateStr.split(' ')[1];

                    el.innerText = `${y}. ${m}. ${d}. (${wDay}) ${timePart}`;
                }
            }
        });

        const urlParams = new URLSearchParams(window.location.search);
        let targetMMDD = urlParams.get('date');

        if (!targetMMDD || !/^\d{2}-\d{2}$/.test(targetMMDD)) {
            const today = new Date();
            const m = String(today.getMonth() + 1).padStart(2, '0');
            const d = String(today.getDate()).padStart(2, '0');
            targetMMDD = `${m}-${d}`;
        }

        const entries = document.querySelectorAll('.diary-entry');
        let visibleCount = 0;

        entries.forEach(entry => {
            const entryMMDD = entry.getAttribute('data-month-day');
            const entryYear = entry.getAttribute('data-year');

            if (entryMMDD === targetMMDD) {
                entry.style.display = 'block';
                visibleCount++;

                if (entryYear === currentRealYear) {
                    // ★ 올해 포스트: 연한 세이지 그린 바탕색 + 맑은 풀잎색 포인트 테두리
                    entry.style.background = '#f0fdf4';
                    entry.style.borderLeft = '4px solid #15803d'; 
                } else {
                    // ★ 지난해(과거) 포스트: 바탕색 없음(흰색) + 세련된 다크 차콜 테두리
                    entry.style.background = '#ffffff';
                    entry.style.borderLeft = '4px solid #334155';
                }
            } else {
                entry.style.display = 'none';
            }
        });

        if (visibleCount === 0) {
            document.getElementById('no-posts-msg').style.display = 'block';
        }
    });
</script>
