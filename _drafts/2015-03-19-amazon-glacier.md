---
layout: post
title: Amazon Glacier for Backups
image: /images/glacier.jpg
date: 2015-03-19 12:00
tag: Amazon Glacier is a super-cheap, flexible backup solution
categories: [backup, administration, configuration, amazon]
---
[1]: http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html
[2]: http://aws.amazon.com/glacier/
[3]: http://docs.aws.amazon.com/cli/latest/userguide/installing.html
[4]: http://docs.aws.amazon.com/cli/latest/userguide/chap-working-with-services.html
[5]: http://docs.aws.amazon.com/cli/latest/reference/glacier/index.html
[6]: http://docs.aws.amazon.com/cli/latest/reference/glacier/list-vaults.html
[7]: http://docs.aws.amazon.com/cli/latest/reference/glacier/initiate-multipart-upload.html

# About Glacier

* Signed up for account
* Created vault


# IAM Console

Open the IAM console.

From the navigation menu, click Users.

Select your IAM user name.

Click User Actions, and then click Manage Access Keys.

Click Create Access Key.

Your keys will look something like this:

Access key ID example: ...

Secret access key example: ...

Click Download Credentials, and store the keys in a secure location.

# Install and Configure CLI

[here][3]

Notice plaintext `~/.aws/credentials`

# No tutorial

[The documentation][4] is a little thin for Glacier.

[More documentation][5] here.

# List vaults

"This operation lists all vaults owned by the calling user's account. The list returned in the response is ASCII-sorted by vault name."

You can [list vaults][6]:

	aws glacier list-vaults --account-id -
	
# Multi-part uploads
[Multi-part][7] uploads are recommended when more than 100 MB is being moved.

	aws glacier initiate-multipart-upload --account-id - --vault-name Pictures --archive-description "Initial upload" --part-size 8388608

Response:

	{
		"uploadId": "...", 
		"location": "..."
	}

Upload a blob:

	