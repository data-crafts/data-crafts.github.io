---
layout: post
title:  "Two Value Engineering tools any Presales can build"
author: abdelkrim
categories: [ Value Engineering, Presales, Management, Process ]
image: assets/images/9-1.jpg
description: "Two Value Engineering tools any Presales can build"
featured: true
hidden: true
toc: true
comments: true
---

Value can be defined as a ratio of function to cost. A new solution can add value by either improving the function or reducing the cost. Value Engineering (VE) in the context of software selling helps measure the value brought by a software when it is used on its own or as part of a larger system. 

In the previous blog, I shared [6 reasons why I think every presales should add VE to their toolbox](https://www.datacrafts.fr/value-engineering-part-1/). In this blog, I’ll share two concrete examples of VE tools that I built in my previous experiences. The main objective of this post is to show you how to easily build powerful tools with a spreadsheet and some basic math. I hope these examples will inspire you to start building your own tools for your industry.

## The TCO Calculator

Whatever domain you are working in, there’s a crazy innovation speed. Every day, a new startup emerges or a new tool launches. Your prospects have a plethora of tools to choose from and they are continuously engaged by your competitors. 

When comparing competing solutions, cost is often a key criteria in your prospects’ mind. They start by shortlisting a few vendors that meet their functional requirements then they compare the cost of these tools. Shortlisted tools may have different characteristics and their costs are hard to compare. Self hosted vs managed, integration vs customization, ease of use vs learning curve are all characteristics that make it hard to do an apples-to-apples comparison. Some of the options may even seem free like OSS tools. How can you win this fight?

Solution Engineers can build a TCO calculator. A TCO calculator measures the holistic cost of a solution including hardware & software costs, employees salaries, release cycle speed and missed opportunity cost, training and other productivity losses. A well-designed TCO calculator should compare the cost of your solution with your competitors’ costs. Finally, it should highlight “the cost of doing nothing” which is the money that your prospect is losing every month while they are using your competitor's solution instead of yours. 

The TCO calculator is my preferred VE tool for three main reasons:
- Easy to build: the majority of data is known or can be easily predicted. Calculations are easy and simple
- Build once, use everyday: the cost of building and running a solution is really similar among companies. If you keep this in mind, you can build a generic tool that can be used for the majority of your opportunities. The ROI and the impact are maximized.
- A deal accelerator: the cost of doing nothing is great for creating a sense of urgency or building a compelling event for your deal when you don’t have a natural one.

Below is an example of how the outcome of a TCO calculator can look like. It shows a drilldown of the different cost drivers evaluated for your  competitor and your products including its different forms: OSS, Self-Managed or SaaS. Having such a tool guarantees that your prospect is doing a fair comparison and doesn’t forget about hidden costs. For instance, the software license of an OSS based solution is 0$ however the TCO is higher than a full fledged SaaS option in this particular scenario. The difference here comes from the FTE cost that’s often overlooked by customers when deciding between a build or buy approach.

![photo]({{site.baseurl}}/assets/images/9-1.png)

To get to this result, you can build a spreadsheet with the following sections:
- **An input tab:** to centralize the collection of all the data you need for your analysis. This tab is splitted into an AS-IS section and a TO-BE State. Examples of data collected here can be data volume, number of users, number of FTE, cost of downtime, number of projects, etc. See an example in the below picture.
- **A hypotheses tab:** to suggest opinionated values that can be used when the prospect doesn’t have the required data. For instance, you can add here the number of FTEs required to build and run your solution based on your real world experience. Or, you can add the unit price of a particular item based on external studies or benchmarks (cost of 1 hour downtime, cost of a server, etc). The input tab uses this data by default, or overwrites it with customer data if they are able to provide it. This is a great way to challenge your customer to do their homework.
- **Sizing tabs:** to estimate the infrastructure and license required. The input of the sizing should be captured in the inputs tab and depends on your space/solution. This is something that you already have in your company. Just reuse it.
- **A pricing tab:** to compute the license/usage cost of your solution based on the output of the sizing tab. Again, this is something your company already has and can be reused.
- **A competitor pricing tab:** to compute your competitor’s cost based on the output of the sizing tab. You can get this data from public information shared on your competitors website.

Below is an example of an input tab. You can notice that it exposes some dynamic options to support several scenarios. This can be implemented using conditional statements or VLookup functions in your spreadsheet tool  

![photo]({{site.baseurl}}/assets/images/9-2.png)

## The business case calculator 

While a TCO calculator is easy to build, it measures value from a cost reduction perspective. This is less transformative than improving the function. A business case calculator measures the improved function and its ROI. That said, it focuses on a particular use case and is by definition less reusable than a TCO calculator. 

The objective here is to show that the additional revenue enabled by your product outweighs the investment cost. The challenge is to translate the improved function into a positive business outcome (PBO). In other words, it’s translating things like “faster queries”, “more accurate predictions” or “higher scalability” into **quantifiable increased revenue** that your prospect will get from using your product. Examples of PBOs include new customer acquisition, new revenue streams or increased customer retention. Here is an example to illustrate these concepts.

I worked with a media/content publisher startup that offered their customers KPIs and analytics on content performance and audience engagement. Their data analytics team reached out to us to learn about our fast visualization analytics at scale. 

Their pain was that they could onboard only a limited number of customers. Delivering a new KPI required delivering a new version of an application in a resource-intensive process deemed frustrating by their customers. As a result, they had to focus on the big whales and had set aside the SMB segment. 

They needed a platform that let any customer easily self-serve on analytics. Understanding the real problem was key to unlock the value conversation. It helped us measure the PBO and build a solid business case. The new analytics platform would allow them to:
- Upsell Enterprise customers to a new pricier plan that offers access to raw data in a powerful and easy-to-use platform. 
- Launch a SaaS product tailored for the smaller customers with self-service analytics by default. 
- Acquire new Enterprise customers thanks to the new features the freed-up engineering capacity could build 

Keeping these PBOs in mind, we drew three scenarios (optimistic, realistic and conservative) and collected hypotheses for each one of them: percentage of customers we can upsell, the price difference between defined plans and the number of monthly new Self Service customers we can acquire. Using this information in a simple spreadsheet proved that the investment is low risk. Even a conservative scenario showed that the new revenue streams will cover for the investment after only one year. A more realistic or optimistic scenario shows a potential of 1 to two 2 dollars additional revenue over 3 years.

![photo]({{site.baseurl}}/assets/images/9-3.png)

What’s next?

For advanced sales opportunities, Value Engineering can help build advanced models or simulations that may require involving a dedicated Value Engineer. But for the majority of use cases, simple math and some excel skills can take your deal to the next level. In this blog we reviewed two examples of Value Engineering tools. The objective was to make the discussion more concrete, not to write a value engineering manual. I purposely focused on easy tools that any Presales can build. In the next blog, I will share some best practices I believe are important to follow to make these tools more impactful.

Thanks for reading me. As always, feedback and suggestions are welcome.

---
Post photo designed by [Eugen Str](https://unsplash.com/@eugen1980) on [Unsplash](https://unsplash.com/)  
