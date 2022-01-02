# buildkite-spot-fleet-scaler

## What is this?

buildkite-spot-fleet-scaler is a script that scales fleet(s) of Buildkite agents up/down as necessary based on the number of queued jobs for each fleet.

## Dependencies

* The AWS CLI (`aws`) command
* Data::Compare (`libdata-compare-perl` on Debian)
* IPC::Run (`libipc-run-perl` on Debian)
* JSON (`libjson-perl` on Debian)
* List::Compare (`liblist-compare-perl` on Debian)
* List::Util (included in Perl core)
* LWP (included in Perl core)

The AWS user used for the script needs to have permissions like the following IAM policy:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "ec2:ModifySpotFleetRequest",
            "Resource": "arn:aws:ec2:*:652694334613:spot-fleet-request/*",
            "Condition": {
                "ForAllValues:StringEquals": {
                    "ec2:ResourceTag/buildkite-scaler-enable": "1"
                }
            }
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeTags",
                "ec2:DescribeSpotFleetRequests"
            ],
            "Resource": "*"
        }
    ]
}
```

An "API Token" must be created on Buildkite that has the "Read Builds" permission.

## How to use it?

This script *only* handles scaling of existing Spot Fleet Requests, you have to create those yourself.

To enable automatic scaling add the following tags to your Spot Fleet Requests:

```
# 'meta-data' parameter from buildkite-agent.cfg
buildkite-agent-meta-data = "queue=default"

# 'spawn' parameter from buildkite-agent.cfg
buildkite-agent-spawn = "1"

# Minimum and maximum number of allowed spot instances.
buildkite-scaler-min-instances = "0"
buildkite-scaler-max-instances = "2"

# Enable the scaler on this fleet.
buildkite-scaler-enable = "1"

# Automatically terminate agents when there is no work (optional).
# NOTE: This is prone to race conditions. I suggest leaving this disabled and designing the agent
# AMIs so that they shut themselves down when idle instead.
buildkite-scaler-terminate-on-scale-down = "1"
```

Set `BUILDKITE_ORGANIZATION` and `BUILDKITE_API_KEY` in the environment and run the `buildkite-spot-fleet-scaler` script with no arguments. The script will update the fleet target capacities as necessary once and then exit. Run it from cron or another part of your automation pipeline.
