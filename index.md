# AI for Trading Card Games

Building AI engines that can play trading card games at a high level.

---

## About

I'm documenting my work building AI agents for trading card games like Magic: The Gathering, Hearthstone, and Lorcana. These games present some of the hardest challenges in game AI: imperfect information, massive action spaces, and complex rules engines.

I'm particularly interested in:
- **AI-assisted coding** for rapidly building game engines
- **Deep reinforcement learning** for creating strong agents through self-play
- **Advanced search methods** for handling imperfect information

---

## Blog Posts

{% for post in site.posts %}
### [{{ post.title }}]({{ post.url }})
*{{ post.date | date: "%B %d, %Y" }}*

{{ post.excerpt }}

[Read more →]({{ post.url }})

---
{% endfor %}

---

*Last updated: October 2025*
