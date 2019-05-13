


We use [gatling](https://gatling.io/) to load test our services.  If you've never used Gatling, you probably need to [install it](https://gatling.io/docs/current/installation/#installation).  For me, I simply downloaded the bundle, unzip'd it and made sure the bin was on my path.

You can test your installation with the out-of-the-box simulations by following the command-line prompts:

```
$ gatling.sh
GATLING_HOME is set to /Users/boneill/tools/gatling
Java HotSpot(TM) 64-Bit Server VM warning: Option AggressiveOpts was deprecated in version 11.0 and will likely be removed in a future release.
Choose a simulation number:
     [0] computerdatabase.BasicSimulation
...
```

After the run completes, it will point you to the reports.

We use [testable.io](http://testable.io) for scale testing.  They provide a number of different means of running scaled tests against services.  One of the more powerful mechanisms is to use gatling.