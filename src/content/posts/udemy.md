---
title: 'Udemy - Bug Disclosed'
published: 2026-01-03
draft: false
tags: ['bug bounty', 'BAC']
toc: true
coverImage:
  src: './lain.gif'
  alt: 'Lain forever'
---
This is a full bug report disclosed because the company hasn't responded to me. It's no longer reproducible since Udemy has disabled the Personal Plan Trial, but the Team Plan Trial is still available. This will probably work as well.

Timeline:
* June 12, 2025 - report submitted
* July 1, 2025 - an H1 triager reproduced my report and changed the status to "program review"
* December 6, 2025 - I messaged that if I didn't get any updates from the program within 7 days, I would proceed with vulnerability disclosure
* December 9, 2025 - got a reply from H1 triager: "Notified program again"

Udemy ignored me again and doesn't consider this a bug. 180 days have already passed, which means I'm free to disclose it. It's a BAC, Medium (5.3) severity that allows you to get paid courses for free.

# Personal Plan allows to enroll in paid courses that should not be accessible without payment
## Summary:
Activation of the Personal Plan Subscription (trial) allows enrollment in the courses that are not listed in the subscription and require separate  payment.

## Steps To Reproduce:
1. Create new account (I used social-login with xxxxxx@gmail.com address)
2. Visit https://www.udemy.com/subscription-checkout/express/?subscription-id=759247 and activate trial subscription
3. Go search  for courses that are not in our plan and require payment  https://www.udemy.com/courses/search/?subs_filter_type=purchasable_only&q=hacking
4. Choose "Ultimate ethical hacking" for example
5. Enable Burp proxy and visit course link https://www.udemy.com/course/ultimate-ethical-hacking/
6. Note that it costs $9.90 and we are offered to proceed to checkout
7. Switch to Burp HTTP history. Look for request https://www.udemy.com/api/2024-01/graphql/ that has in 'badgeClassesByCourseId' in body, note course ID 3581477 in variables section
8. Now send this request to Burp Repeater, replace the request body with new payload and click Send:
```json
{"query":"\n    mutation enrollInCourse($id: ID!) {\n  enrollmentCreate(learningProduct: {id: $id, type: COURSE}) {\n    enrollment {\n      createdAt\n    }\n  }\n}\n    ","variables":{"id":3581477}}
```
9. Just successfully enrolled in a course that have not been paid for
10. To check it, visit  https://www.udemy.com/course-dashboard-redirect/?course_id=3581477 and see a fully functional course
## Impact
* Loss of partial revenue for the company
* Loss of income for instructors
* Massive leaks of paid course content
