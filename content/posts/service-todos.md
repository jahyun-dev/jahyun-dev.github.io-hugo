+++
author = ""
draft = true
date = "2018-04-14T00:27:53+09:00"
title = "서비스 오픈을 앞둔 개발자의 체크리스트"
tags = ["programming"]
categories = ["programming"]
description = "웹/앱 서비스 오픈을 앞두고"
+++

서비스 오픈을 앞두고 고려해야 하는 체크리스트를 모아보았다.

- [ ] SPoF(Single Point of Failure)가 있는가?
- [ ] 서버가 아중화되어 있는가?
- [ ] 성능 측정이 되었는가?
- [ ] CSS, JS의 minify가 되었는가?
- [ ] 
- [ ] Use HTTPS

## Database

## Fault toelerance

- What happens when a dependency starts failing? What if it begins failing slowly?
- How can the system degrade in a graceful manner?
- How does the system react to overload? Is it `well conditioned?`
- What's the worst-case scenario for total failure?
- How quickly can the system recover?
- Is delayable work delayed?
- Is the system as simple as possible?
- How can the system shed load?
- Which failures can be mitigated, and how?
- Which operations may be retried? Are they?

## Scalability

- How does the system grow? WHat is the chief metric with which the system scales?
- How does the system scale to multiple datacenters?
- How does demand vary? How do you ensure the system is always able to handle peak loads?
- How much query processing is done? Is it possible to shape data into queries?
- Is the system replicated?

## Operability

- How can features be turned on or off?
- How do you monitor the system? How do you detect anomalies?
- Does the system have operational needs specific to the application?
- How do you deploy the system? How do you deploy in an emergency?
- What are the capacity needs? How does the system grow?
- How do you configure the system? How do you configure the system quickly?
- Does the system behave in a predictable manner? Where are there nonlinearities in load or failure responses?

## Efficiency

- Is it possible to precompute data?
- Are you doing as little work as possible?
- Is the program as concurrent as possible?
- Does the system make use of work batching?
- Have you profiled the system? Is it possible to profile in situ?
- Are there opportunities for parallelization?
- Can you load test the system? How do you catch performance regressions?


## Performance

## Front-End

## Security

## Global

- Location
- Global Center vs Regional Center
