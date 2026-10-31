# AWS_Rollout_Patching
The script safely performs an in-place patching rollout on the Auto Scaling Group svb-tlz-terraform-enterprise by temporarily increasing capacity from 2 → 4 healthy instances, verifying everything is healthy, then scaling back down to 2. This ensures the service stays available while older instances can be replaced or patched.
1. Configuration & Safety DefaultsAt the top, the script hard-codes the key values:
ASG name: svb-tlz-terraform-enterprise
Target Group: svb-tlz-tfe-443
Region: us-west-2 (overridable)
Scale targets: 4 during the rollout, 2 at the end
Timeout: 15 minutes to reach healthy state
Poll interval: 30 seconds
It also supports a --dry-run flag so you can see exactly what would happen without making any changes.2. Pre-flight ChecksBefore touching anything, the script:
Verifies aws and jq are installed
Looks up the Target Group ARN by name
Confirms the ASG exists and prints its current Desired / Min / Max / instance count
If anything is missing or misconfigured, it exits early with a clear error.3. Core Helper Functionsget_healthy_instance_count

This is the most important safety function. It counts an instance as “healthy” only if both conditions are true:
The instance is in InService state in the ASG
The instance is marked healthy in the Target Group (svb-tlz-tfe-443)
This double-check prevents the script from thinking the service is ready when instances are still registering or failing health checks.wait_for_healthy_count

Polls every 30 seconds until the required number of healthy instances is reached (or the 15-minute timeout is hit).set_scale_in_protection

Turns scale-in protection on or off for all current instances. This is used to lock the instances in place while you verify everything is working.update_asg_capacity

Updates Desired / Min / Max capacity on the ASG (or just prints what it would do in dry-run mode).4. Main Rollout Sequence












































StepWhat HappensWhy1Set Desired = 4, Min = 2, Max = 4Allows the ASG to launch two additional instances2Wait until 4 healthy instances existEnsures the new instances are fully registered and passing health checks3Turn on scale-in protection on all instancesPrevents the ASG from accidentally terminating any instance while you verify4Pause for manual confirmationGives you a chance to check logs, dashboards, or run additional tests before scaling back in5Turn off scale-in protectionAllows the ASG to terminate excess instances6Set Desired = 2ASG terminates the two oldest (or excess) instances7Wait until exactly 2 healthy instances remainConfirms the final state is stable5. Safety Features Built In
Dry-run mode – prints every action without executing it
Double health check (ASG + Target Group)
Scale-in protection during the critical verification window

1. Pre-flight checks pass
2. ASG scales from 2 → 4
3. Script waits (usually 3–8 minutes) until 4 healthy instances are reported
4. All instances get scale-in protection
5. You press Enter after verifying everything looks good
6. Protection is removed
7. ASG scales back to 2
8. Script confirms 2 healthy instances remain and exits successfully
Interactive confirmation before scaling back down (skipped in dry-run)
Timeouts so the script doesn’t hang forever
Clear colored logging with timestamps
