# Alan's Brain Dump on Logs

- [Alan's Brain Dump on Logs](#alans-brain-dump-on-logs)
  - [Prerequisities](#prerequisities)
    - [AWS CLI](#aws-cli)
    - [Filtering and Tools](#filtering-and-tools)
  - [AWS S3 Buckets Logs](#aws-s3-buckets-logs)
      - [Analyze the Logs](#analyze-the-logs)
  - [AWS CloudTrail Logs](#aws-cloudtrail-logs)

## Prerequisities

### AWS CLI
```bash
# You will need your AWS IAM user account access keys and secret before starting this step
brew install awscli

# If you're installing for the first time then it means you need to configure the cli with your profile and credentials

aws configure
```

### Filtering and Tools

**JSON**: 

`brew install jq`

*Fastest JSON parsing library, built in C.* [Read More](https://github.com/stedolan/jq).


**Large Files with Strings**:

`brew install ripgrep`

*Much better than grep. Like really, really fast and it's built in Rust.* [Read More](https://github.com/BurntSushi/ripgrep).


## AWS S3 Buckets Logs
To get all the logs in the bucket, run the following:
```bash
# make a new folder to store the logs
mkdir s3logs

# if you want all the logs in the bucket run 
aws s3 sync s3://<BucketName>/BucketLogs s3logs

# if you want logs for a single date
aws s3 sync s3://<BucketName>/ . --exclude "*" --include "BucketLogs/<YYYY-MM-DD>*"
```

If there is an error message noting that the there are no access keys on the bucket add a `--profile default` flag to the end of the message.

#### Analyze the Logs
Once you have all the logs int he folder, you can use `grep` to search through the logs.

Example `grep` command:
```bash
grep -i 'sql' .
```

If you are trying to filter or cherry pick things from all the logs in the bucket, I suggest you try `ripgrep`, it's a better grep tool built in rust (`brew install ripgrep`), and it could go through 237,000 log files in milliseconds.

Once you get an std output of what you are looking for, you can do have it output the results into a file. (i.e. `grep -i 'sql' > somefilename.txt`) If you want to further analyze or filter stuff you can do `grep -i 'sql' | grep -i 'user/alalee'`

## AWS CloudTrail Logs
Similar to Bucket Logs, you need to actually target the bucket and directory, and download it.

CloudTrail logs will tell you an audit of AWS activity done to resources. The buckets are segregated by `YYYY/MM/DD` and the file output is with a `.gz` extension.

```bash
mkdir awslogs
aws s3 sync s3://<BucketName>/AWSLogs/YYYY/MM/DD awslogs
# this will download all the .gz files
```

Next you have to concatenate all the .gz files because they are essentially partitions of an actual JSON file.
```bash
cat *gz > filenameHere.gz
```
This is the fastest way to concatenate the `.gz` files. Now you just have to unzip it. To prettify the JSON file you can run the following command with jq:

```bash
cat fileName.json | jq . > prettifiedFileName.json
```