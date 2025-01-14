---
title: Contact
nav:
  order: 5
  tooltip: Email, address, and location
---

# {% include icon.html icon="fa-regular fa-envelope" %}Contact

Our lab is looking for highly motivated, passionate, curious, and brave spirits who are dedicated to scientific discovery! We are currently taking graduate students enrolled in the GPiBS Program from both Cell Biology and Neuroscience Departments at OUHSC. Postdocs, research technicians, and undergrad candidates should send an email (with the title "Application_Your Name" to Dr. Zhang expressing their interests, along with a detailed CV and cover letter. Come and join us in our journey to answer the most exciting questions!
{%
  include button.html
  type="email"
  text="Xinxing-Zhang@ouhsc.edu"
  link="Xinxing-Zhang@ouhsc.edu"
%}
{%
  include button.html
  type="phone"
  text="(405) 271-8001"
  link="+1-405-271-8001"
%}
{%
  include button.html
  type="address"
  tooltip="Our location on Google Maps for easy navigation"
  link="https://maps.app.goo.gl/qEdARg1CFQPF2p766"
%}

{% include section.html %}

{% capture col1 %}

{%
  include figure.html
  image="images/photo.jpg"
  caption="Lorem ipsum"
%}

{% endcapture %}

{% capture col2 %}

{%
  include figure.html
  image="images/photo.jpg"
  caption="Lorem ipsum"
%}

{% endcapture %}

{% include cols.html col1=col1 col2=col2 %}

{% include section.html dark=true %}

{% capture col1 %}
Lorem ipsum dolor sit amet  
consectetur adipiscing elit  
sed do eiusmod tempor
{% endcapture %}

{% capture col2 %}
Lorem ipsum dolor sit amet  
consectetur adipiscing elit  
sed do eiusmod tempor
{% endcapture %}

{% capture col3 %}
Lorem ipsum dolor sit amet  
consectetur adipiscing elit  
sed do eiusmod tempor
{% endcapture %}

{% include cols.html col1=col1 col2=col2 col3=col3 %}
