---
layout: home
title: Welcome
---

# Data Science Should Be for Everyone

I’m a data scientist and educator. This site is a place for me to share:
- 🛠️ Projects I've built
- 📓 Posts and ideas about data science education
- 📊 Data science lesson plans, project ideas for students, and more

---

## Featured Personal Projects

Check out [my project showcase](projects.html) for more.

---

## Recent Posts

{% for post in site.posts limit: 3 %}
- [{{ post.title }}]({{ post.url }}) — {{ post.date | date: "%b %-d, %Y" }}
{% endfor %}

[Read the full blog →](blog.html)

---

## Tools for Teachers

Explore some of the [data science lesson plans and projects](lessons.html) I’ve created for high school students.

---
