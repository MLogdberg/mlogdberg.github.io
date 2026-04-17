---
layout: page
title: Videos and Speaking
permalink: /videos
---
<p class="section-intro">A curated set of public videos and sessions featuring Mattias Lögdberg. This page is meant to make it easier to validate experience, watch talks, and get a feel for the topics Mattias speaks about most often.</p>

<p class="section-intro">You can also browse the full public channel on <a href="https://www.youtube.com/@mlogdberg/featured">YouTube</a> for additional uploads and older material.</p>

## Featured

<div class="video-grid">
{% for video in site.data.videos %}
  {% if video.featured %}
  <article class="video-card">
    <span class="video-source">{{ video.source }}</span>
    <span class="video-year">{{ video.year }}</span>
    <h3>{{ video.title }}</h3>
    <p>{{ video.description }}</p>
    <p class="video-why"><strong>Why it matters:</strong> {{ video.why }}</p>
    {% if video.note %}
    <p class="video-note">{{ video.note }}</p>
    {% endif %}
    <a class="video-link" href="{{ video.watch_url }}">{{ video.link_label | default: "Open source page" }}</a>
  </article>
  {% endif %}
{% endfor %}
</div>

## Timeline

<div class="timeline-list">
{% assign timeline_videos = site.data.videos | where_exp: "item", "item.featured != true" | sort: "timeline_date" | reverse %}
{% for video in timeline_videos %}
  <article class="timeline-item">
    <div class="timeline-year">{{ video.date | default: video.year }}</div>
    <div class="timeline-content">
      <span class="video-source">{{ video.source }}</span>
      <h3>{{ video.title }}</h3>
      <p>{{ video.description }}</p>
      <p class="video-why"><strong>Why it matters:</strong> {{ video.why }}</p>
      {% if video.note %}
      <p class="video-note">{{ video.note }}</p>
      {% endif %}
      <a class="video-link" href="{{ video.watch_url }}">{{ video.link_label | default: "Open source page" }}</a>
    </div>
  </article>
{% endfor %}
</div>

## Speaking profile

<p>If you are evaluating Mattias for an event, workshop, or client advisory engagement, the public speaker profile on <a href="https://sessionize.com/mlogdberg/">Sessionize</a> is a good next step. It includes additional session abstracts covering API Management, CI/CD, cloud security, managed identities, and Azure Integration Services.</p>
