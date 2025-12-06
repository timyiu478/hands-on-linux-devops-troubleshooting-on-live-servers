## "Hamburg": Find the AWS EC2 volume

### Problem

Description: We have a lot of AWS EBS volumes, the description of which we have save to a file with: aws ec2 describe-volumes > aws-volumes.json.
One of the volumes contains important data and we need to identify which volume (its ID), but we only remember these characteristics: gp3, created before 31/09/2025 , Size < 64 , Iops < 1500, Throughput > 300.

Find the correct volume and put its "InstanceId" into the ~/mysolution file, e.g.: echo "vol-00000000000000000" > ~/mysolution

Root (sudo) Access: True

Test: Running md5sum /home/admin/mysolution returns e7e34463823bf7e39358bf6bb24336d8 (we also accept the file without a new line at the end).

### Solution

```bash
admin@i-00af238f258a599de:~$ jq -r '.Volumes[] | select(.VolumeType=="gp3") | select(.CreateTime < "2025-09-31T00:00:00.000Z") | select(.Size<64) | select(.Iops<1500) | select(.Throughput>300) ' aws-volumes.json
{
  "AvailabilityZoneId": "use2-az2",
  "Iops": 1000,
  "VolumeType": "gp3",
  "MultiAttachEnabled": false,
  "Throughput": 500,
  "Operator": {
    "Managed": false
  },
  "VolumeId": "vol-29d115ef9c3944f29",
  "Size": 8,
  "SnapshotId": "snap-4d14b6dc50854a9cb",
  "AvailabilityZone": "us-east-2c",
  "State": "in-use",
  "CreateTime": "2025-09-29T16:43:18.004823Z",
  "Attachments": [
    {
      "DeleteOnTermination": true,
      "VolumeId": "vol-29d115ef9c3944f29",
      "InstanceId": "i-371822c092b2470da",
      "Device": "/dev/xvdc",
      "State": "attached",
      "AttachTime": "2025-09-29T17:41:18.004823Z"
    }
  ],
  "Encrypted": false
}
{
  "AvailabilityZoneId": "use2-az3",
  "Iops": 1000,
  "VolumeType": "gp3",
  "MultiAttachEnabled": false,
  "Throughput": 500,
  "Operator": {
    "Managed": false
  },
  "VolumeId": "vol-99646e602c6e4b92a",
  "Size": 16,
  "SnapshotId": "snap-27b0fb199d294889b",
  "AvailabilityZone": "us-east-2a",
  "State": "available",
  "CreateTime": "2025-09-29T01:21:18.004823Z",
  "Attachments": [],
  "Encrypted": false
}
admin@i-00af238f258a599de:~$ echo "i-371822c092b2470da" > mysolution
admin@i-00af238f258a599de:~$ ./agent/check.sh 
OKadmin@i-00af238f258a599de:~$ 
```
