# Troubleshooting the Elastic CI Stack for AWS

<!-- alex ignore easy -->

Infrastructure as code isn't always easy to troubleshoot, but here are some ways to debug exactly what's going on inside the [Elastic CI Stack for AWS](https://github.com/buildkite/elastic-ci-stack-for-aws), and some solutions for specific situations.

## Using CloudWatch Logs

Elastic CI Stack for AWS sends logs to various CloudWatch log streams:

* Buildkite Agent logs get sent to the `buildkite/buildkite-agent/{instance_id}` log stream. If there are problems within the agent itself, the agent logs should help diagnose.
* Output from an Elastic CI Stack for AWS instance's startup script ([Linux](https://github.com/buildkite/elastic-ci-stack-for-aws/blob/master/packer/linux/conf/bin/bk-install-elastic-stack.sh) or [Windows](https://github.com/buildkite/elastic-ci-stack-for-aws/blob/master/packer/windows/conf/bin/bk-install-elastic-stack.ps1)) get sent to the `/buildkite/elastic-stack/{instance_id}` log stream. If an instance is failing to launch cleanly, it's often a problem with the startup script, making this log stream especially useful for debugging problems with the Elastic CI Stack for AWS.

Additionally, on Linux instances only:

* Docker Daemon logs get sent to the `/buildkite/docker-daemon/{instance_id}` log stream. If docker is having a bad day on your machine, look here.
* Output from the cloud init process, up until the startup script is initialised, is sent to `/buildkite/cloud-init/output/{instance_id}`. Logs from this stream can be useful for inspecting what environment variables were sent to the startup script.

On Windows instances only:

* Logs from the UserData execution process (similar to the `/buildkite/cloud-init/output` group above) are sent to the `/buildkite/EC2Launch/UserdataExecution/{instance_id}` log stream.

There are a couple of other log groups that the Elastic CI Stack for AWS sends logs to, but their use cases are pretty specific. For a full accounting of what logs are sent to CloudWatch, see the config for [Linux](https://github.com/buildkite/elastic-ci-stack-for-aws/blob/master/packer/linux/conf/cloudwatch-agent/config.json) and [Windows](https://github.com/buildkite/elastic-ci-stack-for-aws/blob/master/packer/windows/conf/cloudwatch-agent/amazon-cloudwatch-agent.json).

## Accessing  Elastic CI Stack for AWS instances directly

Sometimes, looking at the logs isn't enough to figure out what's going on in your instances. In these cases, it can be useful to access the shell on the instance directly:

* If your Elastic CI Stack for AWS has been configured to allow SSH access (using the `AuthorizedUsersUrl` parameter), run `ssh <some instance id>` in your terminal
* If SSH access isn't available, you can still use AWS SSM to remotely access the instance by finding the instance ID, and then running `aws ssm start-session --target <instance id>`

## Auto Scaling group fails to boot instances

Resource shortage can cause this issue. See the Auto Scaling group's Activity log for diagnostics.

To fix this issue, change or add more instance types to the `InstanceType` template parameter. If 100% of your existing instances are Spot Instances, switch some of them to On-Demand Instances by setting `OnDemandPercentage` parameter to a value above zero.

## Stacks over-provision agents

If you have multiple stacks, check that they listen to unique queues—determined by the `BuildkiteQueue` parameter. Each Elastic CI Stack for AWS you launch through CloudFormation should have a unique value for this parameter. Otherwise, each scales out independently to service all the jobs on the queue, but the jobs will be distributed amongst them. This will mean that your stacks are over-provisioned.

This could also happen if you have agents that are not part of an Elastic CI Stack for AWS [started with a tag](/docs/agent/v3/cli-start#tags) of the form `queue=<name of queue>`. Any agents started like this will compete with a stack for jobs, but the stack will scale out as if this competition did not exist.

## Instances fail to boot Buildkite Agent

See the Auto Scaling group's Activity logs and CloudWatch Logs for the booting instances to determine the issue. Observe where in the `UserData` script the boot is failing. Identify what resource is failing when the instances are attempting to use it, and fix that issue.

## Instances fail jobs

Successfully booted instances can fail jobs for numerous reasons. A frequent source of issues is their disk filling up before the hourly cron job fixes it or terminates them.

An instance with a full disk can be causing jobs to fail. If such instance is not being replaced automatically — for example, because of a stack with the `MinSize` parameter greater than zero, you can manually terminate the instance using the EC2 Dashboard.

## Permission errors when running Docker images with volume mounts

The Docker daemon is configured by default to run containers in a [username namespace](https://docs.docker.com/engine/security/userns-remap/). This will map the `root:root` user and group inside the container to the `buildkite-agent:docker` on the host. You can disable this using the stack parameter `EnableDockerUserNamespaceRemap`.
