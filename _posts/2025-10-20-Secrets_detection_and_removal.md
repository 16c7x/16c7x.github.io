---
title: Secrets detection and removal
date: 2025-10-20 11:12:00 -0000
categories: [Gitlab]
tags: [Gitlab, Secrets, Security, Git]
---

# Secrets detection and removal

## Secrets scanning policy

Scan execution policies can be setup at the group level, it's common for something like a secrets detection to be setup at the top level so it is applied globally. To enable, in the group where the policy is required go to **Secure -> Policies** and select **New policy**.
Select **Scan policy**, most of the defaults can be left, just give it a name and tick **Secret Detection** check box. 
The secret scan will be applied to all merge requests.


## Secrets removal

### Removing secrets demo.

Create a new project called secrets-test and clone it, create new branch called dev, add a file and push.

```
git checkout -b dev
echo "info file" > info.txt
git add info.txt
git commit -m "new info file"
git push origin dev
```

In Gitlab, create a merge request to merge dev into main, The scan pipeline should trigger and the scan results will be displayed with no problems. 

Back in the project, add a secret and push it.

```
echo "Commit created with glpat-12312312312312312312 token" >> info.txt
git add info.txt
git commit -m "added a secret"
git push origin dev
```

Back on Gitlab, the scan should trigger again in the merge request, but this time with a vulnerability. 

Back in the project, remove the secret and push, this is the most likely first attempt at removing a secret. 

```
vi info.txt
git add info.txt
git commit -m "remove a secret"
git push origin dev
```

Back on Gitlab, the scan should trigger again in the merge request, the secret is still in the history so it should fail. The project is going to need the secret cleaned from it's history.

```
git log
git rebase -i <id of the commit with the secret>
```

Change the **pick** to **edit** on the commit where the secret was added. Now edit the info.txt file to remove the secret, update the commit and push.

```
git add info.txt
git commit --amend --allow-empty
git rebase --continue
git push origin dev --force-with-lease
```

Check back on Gitlab, the security scan should now be clean again and the merge can be completed.

## Push Protection

Obviously, the best policy would be to ensure secrets did not make it into your Gitlab repo in the first place. This can be implemented using **Secrets push protection**. This can be enabled at the project level **Secure -> Security configuration** and enabling the **Secret push protection** radio button.

### Push protection demo

```
echo "Commit created with glpat-12312312312312312312 token" >> info.txt
git add info.txt
git commit -m "put the secret back"
git push origin dev
```

The commit command will return an error saying the commit has been rejected. 

It's important to remember that if the secret already exists in the file, it will not be blocked if pushed again unless the secret changes after the push protection is enabled. 