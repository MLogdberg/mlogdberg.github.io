---
layout: page
title: About Mattias Lögdberg
permalink: /about
---
<div class="profile-intro">
  <div class="profile-copy">
    <p><strong>Mattias Lögdberg is an Azure integration, cloud security, and platform architecture specialist.</strong></p>

    <p>My work has long focused on helping teams build secure and reliable cloud solutions, with a special focus on Azure Integration Services, API Management, networking, identity, and practical delivery in enterprise environments.</p>

    <p>Today I work across architecture, implementation, advisory, and education. That means everything from shaping integration platforms and landing zones to helping teams improve developer experience, delivery quality, and day-to-day decision making in Azure. My experience in the Microsoft Azure integration space, together with a strong passion for sharing knowledge, has led to me being awarded Microsoft MVP since 2017.</p>

    <p>I also run the Integration User Group in Sweden and regularly speak at conferences, community events, and podcasts. If you have used something I have written, seen one of my sessions, or just want to talk integration and cloud security, feel free to reach out.</p>
  </div>
  <div class="profile-photo">
    <a href="/assets/uploads/img/picture.jpg">
      <img src="/assets/uploads/img/picture.jpg" alt="Mattias Lögdberg">
    </a>
  </div>
</div>

## Featured videos

<p class="section-intro">A few good starting points if you want to hear how I think about Azure integration, identity, networking, and cloud security.</p>

<div class="video-grid">
{% for video in site.data.videos %}
{% if video.featured %}
  <article class="video-card">
    <span class="video-source">{{ video.source }}</span>
    <h3>{{ video.title }}</h3>
    <p>{{ video.description }}</p>
    <a class="video-link" href="{{ video.watch_url }}">Watch video</a>
  </article>
{% endif %}
{% endfor %}
</div>

<p><a class="btn" href="/videos">See all videos and speaking appearances</a></p>
