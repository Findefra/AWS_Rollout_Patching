# AWS_Rollout_Patching
The script safely performs an in-place patching rollout on the Auto Scaling Group svb-tlz-terraform-enterprise by temporarily increasing capacity from 2 → 4 healthy instances, verifying everything is healthy, then scaling back down to 2. This ensures the service stays available while older instances can be replaced or patched.
