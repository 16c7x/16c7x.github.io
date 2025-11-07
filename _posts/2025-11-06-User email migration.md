---
title: User email migration
date: 2025-11-06 11:12:00 -0000
categories: [Gitlab]
tags: [Gitlab, Users, Email, Accounts]
---

# User email migration

## The problem 

You may find yourself in a situation where all of your users email addresses need to change. This can happen if your organization is acquired by another or if your organization decides to go through a total rename and rebranding exercise. How do you change all of your Gitlab users email addresses, and how, as Gitlab uses users email addresses to link them to their history, do you maintain that link after the address change. 

## The solution

Gitlab maintains the relationship with things such as commits using a users email address, a user can have multiple email addresses and we can use this to maintain that link to their history. We demote the users old email address to a secondary email address, this maintains their history, then add a new primary email address.

## Migrating users email addresses

We can demote the email address to a secondary address and add the new address with a rake task.

### The data

We need is some data, the users old email address and their new one in a csv file.

```
# /tmp/email_updates.csv
user1@example.com,user1@acme.com
user2@example.com,user2@acme.com
```


### The script

Then a rake task that will read that data, find the user using their primary email address, demote the primary address to a secondary and add the new primary address.

Copy the rake task below to ```/opt/gitlab/embedded/service/gitlab-rails/lib/tasks/custom_user_task.rake```

```
require 'csv'

namespace :custom do
  desc "Update user emails from CSV file"
  task :update_user_emails_from_csv, [:csv_file_path] => :environment do |t, args|
    csv_file_path = args[:csv_file_path]

    if csv_file_path.nil?
      puts "Usage: gitlab-rake custom:update_user_emails_from_csv[/path/to/file.csv]"
      puts "CSV format: old_email,new_email (one pair per line)"
      exit 1
    end

    unless File.exist?(csv_file_path)
      puts "Error: File '#{csv_file_path}' not found"
      exit 1
    end

    begin
      successful_updates = 0
      failed_updates = 0

      CSV.foreach(csv_file_path) do |row|
        old_email = row[0]&.strip
        new_email = row[1]&.strip

        if old_email.nil? || new_email.nil? || old_email.empty? || new_email.empty?
          puts "Skipping invalid row: #{row.inspect}"
          failed_updates += 1
          next
        end

        puts "\nProcessing: #{old_email} -> #{new_email}"

        user = User.find_by(email: old_email)

        if user
          puts "  Current email: #{user.email}"
          user.update!(email: new_email)
          user.confirm
          user.save!
          puts "  New email: #{user.email}"
          puts "  ✓ Successfully updated!"
          successful_updates += 1
        else
          puts "  ✗ User with email '#{old_email}' not found"
          failed_updates += 1
        end
      end

      puts "\n=== Summary ==="
      puts "Successfully updated: #{successful_updates}"
      puts "Failed updates: #{failed_updates}"

    rescue => e
      puts "Error processing CSV: #{e.message}"
    end
  end
end
```

And then run the task ```sudo gitlab-rake custom:update_user_emails_from_csv[/tmp/email_updates.csv]```

## What if a user deletes their old email

But what happens if the user accidentally deletes their secondary address, they can't just add it back in because Gitlab will want them to verify it, and they probably don't have access to that mail box anymore. We can fix that with another rake task to confirm that email address without using the link in the email.

```
sudo gitlab-rails console
user = User.find_by(email: '<email address>')
email = user.emails.find_by(email: '<email address>')
email.confirm
```