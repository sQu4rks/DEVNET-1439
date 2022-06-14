# Continuous monitoring using pyATS test cases

pyATS testcases, when written correctly, contain the ability to test whether or not our network is in the shape we want it to be in. So why only use this when applying changes? We can run our test cases continuously and have the results be send to us in a chat space without writing a single line of code. 

For this demo to work you'll need the example testcase for BGP from demo 3 and also a bot token for Webex. You can go to [developer.webex.com](developer.webex.com/) to create such a bot token. 

1. Assuming you have the same directory structure as in demo 3, you can have pyATS send you a notification by simply ammending two arguments, `--webex-token` and `--webex-email` to the run command. The first specifies the bot token and the second specifies the users webex e-mail that the message should be send to. The new run command now looks like this:
```
pyats run job bgp_check_job.py --testbed-file cml-testbed-lab.yaml --webex-token <Webex TOKEN> --webex-email <Your e-mail>
```

That's it! Wait for the run to finish and you'll receive a summary of your test via Webex.

<div align="right">
   
   [Previous](../04-parallel_config/) - Next
</div>