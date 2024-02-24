Postmortem

Following the rollout of Holberton School's System Engineering & DevOps project 0x19, an incident unfolded around 00:07 Pacific Standard Time (PST) within an isolated Ubuntu 14.04 container housing an Apache web server. Unexpectedly, GET requests to the server began returning 500 Internal Server Errors instead of the anticipated response, which should have been an HTML file defining a straightforward Holberton WordPress site.

Debugging Process
Our intrepid bug hunter Brennan (dubbed BDB for brevity) encountered this issue shortly after 19:20 PST, promptly diving into the fray to resolve it. His investigation commenced with a check on the running processes using the ps aux command, confirming that both root and www-data apache2 processes were operational.

Further examination led Brennan to the sites-available folder within the /etc/apache2/ directory, affirming that the web server was indeed configured to serve content from the /var/www/html/ directory. Despite this, running strace on the PID of the root Apache process and subsequently curling the server failed to yield any useful insights.

Undeterred, Brennan repeated the process with the www-data process, this time hitting the jackpot. Strace unveiled an -1 ENOENT (No such file or directory) error attempting to access a file named /var/www/html/wp-includes/class-wp-locale.phpp.

Armed with this discovery, Brennan meticulously combed through files in the /var/www/html/ directory, utilizing Vim pattern matching to pinpoint the erroneous .phpp file extension within the wp-settings.php file, specifically on Line 137.

With surgical precision, Brennan excised the extraneous 'p' from the file extension, swiftly resolving the issue. A subsequent curl test confirmed the success of the fix.

Summation
In essence, the root cause of the outage boiled down to a simple typo. Specifically, the WordPress application encountered a critical error in wp-settings.php while attempting to load the file class-wp-locale.phpp, when it should have been seeking class-wp-locale.php located in the wp-content directory.

The remedy involved a straightforward correction of the typo, removing the trailing 'p' from the file extension.

Prevention
To prevent similar outages in the future, rigorous testing of applications before deployment is paramount. Moreover, implementing robust status monitoring solutions, such as UptimeRobot, can provide instant alerts in the event of website downtime.

In response to this incident, Brennan crafted a Puppet manifest (0-strace_is_your_friend.pp) to automate the resolution of identical errors in the future, ensuring swift mitigation should they recur. This manifest replaces any 'phpp' extensions in the file /var/www/html/wp-settings.php with 'php'.






