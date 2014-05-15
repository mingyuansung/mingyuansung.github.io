---
layout: post
title: Some useful note at work
---


<p>php sf2/app/console doctrine:migrations:status<br>
php sf2/app/console doctrine:migrations:diff   (to create a doctrine file)<br>
php sf2/app/console doctrine:migrations:execute —up 20140513142030<br>
php sf2/app/console doctrine:migrations:execute —down 20140513142030</p>

<p>git push -f ming msung (push work to ming repo, msung branch)<br>
git push -f si msung (force push work to si repo msung branch)<br>
git log<br>
git commit —amend (commit work reusing previous commit, so only one commit shows up)<br>
git rebase -i HEAD~3 (interactive mode rebase to 3 revisions before)<br>
git rebase —abort<br>
fir fetch si<br>
git rebase si/master  (update current working branch to si repo master branch)<br>
git push si msung (update my branch repo)<br>
git rebase -i msung</p>

<pre><code>• Install PHP mongo driver manually if you do not want the latest version using Macport.  http://www.w3resource.com/mongodb/install-php-driver.php
• Use port 23 for access s44-buildbox-1 to download SQL backup file.  If not specify port, the connection will go t port 70 or 22 and time out.
• After install less, need to configure the node_module location path in the parameter yml file.  Mine is Node_paths: [/opt/local/lib/node_modules]
• If ‘sudo rabbitmq-plugins enable rabbitmq_management’ fails complaining about enlarge.cookie file access, do ‘chmod 600 .erlang.cookie’ should resolve it.
• Rabbitmq needs an earlier erlang install since it does not recognize erlang v17.  So ‘sudo port uninstall erlang’ first, then go to https://www.erlang-solutions.com/downloads/download-erlang-otp and pick maybe R16B03, download the dmg installation package and install it.
• Download from http://www.rabbitmq.com/install-generic-unix.html and untar the file, move to /usr/local directory.
• Do a ‘sudo sbin/rabbitmq-server’ to start the server, then use your browser to go to http://localhost:15672/ then login using guest/guest to confirm your installation.
</code></pre>

