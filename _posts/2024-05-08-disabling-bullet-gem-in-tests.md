# Disabling Bullet Gem in Tests. Why?

[The Bullet gem](https://github.com/flyerhzm/bullet) is a handy tool in Rails development, alerting developers to potential N+1 query issues that could impact application performance. However, there are instances where disabling Bullet in tests might actually be beneficial rather than detrimental :) 

## Spotting false alerts :flashlight:
One common scenario where this arises is when Bullet raises alerts about N+1 queries that are known to be false. This can occur due to the minimal test data used, that is not accurately reflecting the behavior of real-life user interaction. In other words, when the test case is there to ensure the functionality of an isolated part of the application, especially if you’ve added the gem to your test environment at a later stage.


## Fork in the road :crossed_swords:
In such cases, developers may find themselves in a dilemma: either spend time updating tests to include more comprehensive setup, or disable Bullet and continue with the existing test suite setup.


## But at what cost? :moneybag:
```
  Bullet.enable = false
```

It's important to note that disabling Bullet in tests should not be your go-to choice always. Rather, it should be implemented with great care, based on the application's behavior and performance requirements, considering the resources you have.

Updating tests to provide more '_accurate_' data can certainly address the issue flagged by Bullet. However, this approach often comes with a big-time trade-off: increased test setup time. Adding more setup steps to tests can significantly slow down the test suite, making the development process less efficient and consuming developers’ valuable time. Moreover, some of these additional setup steps may not directly contribute to testing the functionality under examination, adding unnecessary complexity to the test codebase.

## A car? :racing_car:
Additionally, periodic reviews of Bullet alerts in integration tests can help ensure that N+1 query issues are not overlooked. This means that if you are not obliged to keep the Bullet gem in your environment all the time. Treat your application like a car that needs to be examined periodically. This is an another way to look at the development process.

## Last words 
While the Bullet gem provides valuable insights into potential performance bottlenecks, there are situations where disabling it in tests can be a practical choice. By striking a balance between test efficiency and accuracy, one can improve their development workflow while still maintaining a peace of mind :brain:
