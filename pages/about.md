---
layout: page
title: About
permalink: /about/
weight: 3
---

# **About Me**

Hi my name is **{{ site.author.name }}** :wave:,<br>
Welcome to my blog, I would love to share with you all of the precious lessons I 've been gathering and learning from a different sources (training websites, youtube, other blogs, my co-workers, from projects that I work on, last but not least from so please hit me up on github for any feedback...) What we are trying to achieve is to write clean code,easy to read and easy to maintain, in another word they will be adaptive. It might sound simple but I think it will take a lot of effort to master this craft...I am still learning...:smile: 


<div class="row">
{% include about/skills.html title="Programming Skills" source=site.data.programming-skills %}
{% include about/skills.html title="Other Skills" source=site.data.other-skills %}
</div>

<div class="row">
{% include about/timeline.html %}
</div>
