### VerbaVinculum Analysis

**Table of Contents**
- [Introduction](#introduction)
- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Exploratory Data Analysis and Methodology](#exploratory-data-analysis-and-methodology)
- [Insights](#insights)
- [Recommendations](#recommendations)

## Introduction
The VerbaVinculum app, a burgeoning platform in the social media landscape, has recently undergone a comprehensive performance analysis. This initiative, spearheaded by the new management and administration, aimed to dissect various facets of the app’s performance to gauge its current wellbeing and streamline future decision-making processes.

## Project Overview
This analysis project aims to provide insights on three primary objectives among others, namely;
- Evaluate the app’s user engagement metrics and growth patterns.
- Assess the effectiveness of current features and identify areas for enhancement.
- Investigate the app’s technical stability and scalability prospects.

## Data Sources
Six primary datasets were used to aid in this analysis, namely; 
- users: This contains detailed information on users and their specific IDs.
- likes:  This contains detailed information on photos and which users liked the photos.
- photos:  This contains detailed information on photo IDs for each one posted together with a specific image unified resource collector (URL).
- comments:  This contains detailed information on what users commented on photos that had been posted. 
- tags:  This contains detailed information on hashtag name and specific IDs to each.
- photo_tags:  This contains detailed information on hashtags that were posted alongside each photo.

**DISCLAIMER**: The datasets were all curated information was filled in accordingly. For that reason, no prior cleaning and transformation was done as all the data was well organised, already transformed and ready for analysis, hence all results produced are of high accuracy. 

## Exploratory Data Analysis and Methodology
The business stakeholders were simply eager to have a general insight on how the business was doing and a variety of the questions were aired out. That aside, additional exploration was done as well. Below are some of the questions that were asked and the methodologies that were used to derive the accurate responses;
- Which users have never posted a single photo yet? This would help easily tagregt inactive users with an email campaign, for example.
```sql
SELECT username 
FROM users
LEFT JOIN photos 
ON users.id = photos.user_id
WHERE photos.id IS NULL
```

- Based on the current data, which user got the most likes on a single post? One particular stakeholder wanted to run a contest to know who had the highest likes and congratulate him/ her.
```sql
 SELECT users.username, photos.id, photos.image_url, COUNT(*) AS total_likes
 FROM likes 
 INNER JOIN photos ON likes.photo_id = photos.id 
 INNER JOIN users ON users.id = likes.user_id 
 GROUP BY 1
 ORDER BY total_likes DESC 
 LIMIT 1
```

- How many times does the average user post a photo?
```sql
SELECT ROUND((SELECT COUNT(*) FROM photos)/(SELECT COUNT(*) FROM users),2) AS no_of_avg_posters
```

- What is the total number of posts for all users on the platform so far?
 ```sql
   SELECT SUM(sub.user_count) total_postings
 FROM (SELECT users.username, COUNT(photos.image_url) as user_count
FROM users
JOIN photos ON users.id = photos.user_id 
GROUP BY 1
ORDER BY 2 DESC) sub 
```

- Many posts were noticed to have hashtags used. What are the top 5 most famous and popular hashtags?
```sql
SELECT tag_name, COUNT(tag_name) as no_of_tags
FROM tags
JOIN photo_tags ON tags.id = photo_tags.tag_id
GROUP BY tags.id
ORDER BY 2 DESC 
LIMIT 5
```

- Identify users that have liked every single photo on the platform. This would help identify bots (fake or unreal accounts or users).
```sql
SELECT users.id, users.username, COUNT(users.id) as total_likes_per_user
FROM users
JOIN likes ON users.id = likes.user_id
GROUP BY users.id
HAVING total_likes_per_user = (SELECT COUNT(*) FROM photos)
```
- To have a greater understanding of the platform currently, find the percentage of users who have either never commented on a photo or have liked every photo.
```sql
SELECT sub1.total_sub1 AS never_commented, 
	   (sub1.total_sub1/(SELECT COUNT(*) FROM users))*100 AS perc_no_comment,
		sub2.total_sub2 AS liked_all_photos,
		(sub2.total_sub2/(SELECT COUNT(*) FROM users))*100 AS perc_liked_all
FROM (
		SELECT COUNT(*) AS total_sub1
		FROM(SELECT users.username, comments.comment_text
			FROM users 
			LEFT JOIN comments ON users.id = comments.user_id
			GROUP BY users.id
			HAVING comments.comment_text IS NULL ) AS total_no_of_no_comments
	  ) AS sub1
	  JOIN 
	  (
		SELECT COUNT(*) AS total_sub2
		FROM (SELECT users.id, users.username, COUNT(users.id) AS liked_every_photo
			FROM users
			JOIN likes ON users.id = likes.user_id
			GROUP BY users.id
			HAVING liked_every_photo = (SELECT COUNT(*) FROM photos)) AS total_number_of_all_liked
	  ) AS sub2
```

## Insights
- **User Engagement**: The frequency of posts per user exceeded expectations, with an average of 2.0 posts compared to the anticipated 1.5.
- **Content Creation**: The total number of user-generated posts has reached 257, indicating a healthy level of activity and engagement on the platform.
- **Trending Topics**: Popular hashtags that resonate with our user base include #smile, #beach, #party, #fun, and #concert, showcasing a diverse range of interests and moments shared.
- **Bot Activity**: A small subset of accounts, totaling 13, were identified as bots. This highlights the importance of implementing robust measures to maintain the authenticity of interactions within the app.
- **User Interaction**: Analysis of user interaction patterns revealed that 23% of users have never commented on a photo, while 13% have liked every photo. These statistics provide valuable insights into user behaviors and potential areas to foster more engagement.
These insights will be instrumental in guiding strategic decisions to enhance user experience and platform performance.

## Recommendations
- **Enhance User Engagement Strategies**: To capitalize on the higher-than-expected posting frequency, develop targeted campaigns that encourage users to share content more frequently, potentially increasing overall engagement.
- **Content Creation Incentives**: With a growing number of posts, consider introducing features that reward user creativity, such as content creator spotlights or post-of-the-day awards, to sustain and boost content generation.
- **Hashtag Promotion**: Leverage the popularity of the top hashtags by creating themed events or challenges around #smile, #beach, #party, #fun, and #concert to drive community participation and content diversity.
- **Bot Detection and Management**: Implement advanced algorithms for bot detection and enhance verification processes to preserve the integrity of user interactions and data reliability.
- **Encourage Diverse Interactions**: Address the passive user behavior by introducing features that encourage commenting, such as comment rewards or highlighting insightful comments, and balance the liking behavior by setting limits or diminishing returns on excessive likes to promote genuine interactions.
These recommendations aim to improve the overall health of the app, foster a vibrant community, and guide the platform towards a more engaged and authentic user experience.
