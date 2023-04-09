# System Design

## Step by Step
- Ask about functional requirements, such as
  - Another function like comment or like is needed?
- Ask about non functional requirements, like:
  - high availability: describes systems that are dependable enough to operate continuously without failing. 
  - Consistent: data is in a consistent state when a transaction starts and when it ends. for the example eventual consistent like: when upload video to youtube might be not available real time.

## Tips
- Don't go too detail

## Feed Generation like facebook home or twitter timeline
<image src="assets/feed-generation.png" width="500"/>

### Naive implementation (Fan-out read)
``` sql
SELECT FeedItemID FROM FeedItem WHERE SourceID in ( SELECT EntityOrFriendID FROM UserFollow WHERE UserID = <current_user_id> ) ORDER BY CreationDate DESC LIMIT 100
```

#### Issues of this naive implementation:
Super slow for users with a lot of friends/follows as we have to perform sorting/merging/ranking o a huge number of posts

We generate the timeline when a user loads their page. This could be quite slow and have high latency.

For live updates, each status update will result in feed updates for all followers. This could result in high backlogs in our Newsfeed Generation Service.

To improve the efficiency, we can pre-generate the timeline and store it in a memory.

#### Fanout pull
When you request for news feed, you creates a read request to the system. With fanout read, the read request is fanned out to all your followees to read their posts
<image src="assets/fanout_pull.png" />

#### Fanout push
When you send a new post, you creates a write request to the system. With fanout write, the write request is fanned out to all your followers to update their newsfeed.
<image src="assets/fanout_push.png" />

#### Hybrid
issue of fanout push is when user has large followers and issue in fanout pull is when user has large following, that's why hybrid comes out, if let's say people that have more than 10K followers will go to fanout_pull otherwise go to fanout_push.

## Ecommerce System Design
<image src="assets/ecommerce_system_design.png" width="500"/>

## Yutube System Design
<image src="assets/youtube.png" width="500"/>

## Next
Source: https://github.com/karanpratapsingh/system-design 
- Chapter I
- Chapter II
- Chapter III
- Chapter IV
- Chapter V
Source: https://github.com/weeeBox/mobile-system-design
- All
- Glide with dynamic width and height image from URL

## References
- https://leetcode.com/discuss/interview-question/system-design/3257508/how-does-live-streaming-works
- 
