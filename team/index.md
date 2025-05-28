---
title: Team
nav:
  order: 3
  tooltip: About our team
---

# {% include icon.html icon="fa-solid fa-users" %}Team

Our lab is made up of a group of extremely dedicated and synergistic researchers.  We believe that science, and research, is a 
bridge that connects a myriad of cultures, ideas, and beliefs.  We are committed to cultivating a lab culture that prides itself 
on its inclusivity and diverse team, and where are differences only make us stronger.

{% include section.html %}

{% include list.html data="members" component="portrait" filter="role == 'pi'" %}
{% include list.html data="members" component="portrait" filter="role != 'pi'" %}

{% include section.html background="images/background.jpg" dark=true %}

If you want to find out more about our research, current projects, and team members get in touch with us! We encourage anyone who has an interest in who we are and what we do, feel free to reach out!

{% include section.html %}

{% include section.html %}
## Alumni
{% include list.html data="alumni" component="portrait" filter="role == 'alumni'" %}
{% include list.html data="alumni" component="portrait" filter="role != 'alumni'" %}
