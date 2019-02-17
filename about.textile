---
layout: page
title: About
header: About Me
---

<div>
    <h6>
        <ul class="list-unstyled list-inline">
            <li>
                Links
            </li>
            {% if site.author.github %}
            <li>
                <a href="https://github.com/{{ site.author.github }}">
                    Github <i class="fa fa-github-alt fa-lg"></i>
                </a>
            </li>
            {% endif %}
            {% if site.author.linkedin %}
            <li>
                <a href="https://linkedin.com/in/{{ site.author.linkedin }}">
                    LinkedIn <i class="fa fa-linkedin fa-lg"></i>
                </a>
            </li>
            {% endif %}
            {% if site.author.email %}
            <li>
                <a href="mailto:{{ site.author.email }}">
                    Email <i class="fa fa-envelope"></i>
                </a>
            </li>
            {% endif %}
            {% if site.resume_path %}
            <li>
                <a href="{{ site.BASE_PATH }}/{{ site.resume_path }}">
                    Résumé
                </a>
            </li>
            {% endif %}
        </ul>
    </h6>
</div>

<p>
<img style="float:right; padding: 0px 0px 20px 20px;" src="{{ site.BASE_PATH }}/assets/media/about-me-pic.png">

I was raised in northeast Ohio. After obtaining my BSCS in Computer Science from Ohio University I quickly landed in Atlanta, where I now reside. My main interests include algorithms, neural networks, distributed systems, programming patterns, and performance optimization. I also enjoy getting pedantic about clean, elegant code and API design. My tools of choice include python, C#, Solr, Zookeeper, Jmeter, and C++. However, I generally enjoy playing around with a variety of technologies.

I'm currently a consultant with Sogeti USA - a division of Capgemeni. I focus on performance optimization, distributed system architecture, and C# development.
</p>
