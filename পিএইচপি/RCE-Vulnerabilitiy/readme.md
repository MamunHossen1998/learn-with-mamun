### Laravel + Docker প্রোডাকশনে Vulnerability Check, Patch ও Verification গাইড
 Laravel Application, PHP-FPM, Nginx, MySQL, Queue Worker, Scheduler এবং Reverb থাকলে শুধু Laravel-এর Version চেক করলেই হবে না। 
 PHP Runtime, Composer Dependencies, Docker Base Image, Operating System Packages, Nginx এবং Deployment Configuration—সবগুলো পরীক্ষা করতে হবে।
### কোন কোন স্তরে vulnerability থাকতে পারে
| স্তর |	উদাহরণ | চেক করার টুল |
|---|---|---|
| Composer প্যাকেজ | laravel/framework, guzzle, league/*	| composer audit, Trivy |
| NPM প্যাকেজ	| vite, axios, ইত্যাদি | npm audit, Trivy |
| Docker base image / OS প্যাকেজ |	php:8.x-fpm, nginx, alpine/debian |	Trivy, Docker Scout, Grype |
| PHP runtime নিজে |	PHP ভার্সন (EOL বা পুরনো) |	php -v + php.net supported versions |
| অ্যাপ কনফিগারেশন	| APP_DEBUG=true, .env expose	| ম্যানুয়াল চেক, Enlightn |
| Docker/Host কনফিগ |	root ইউজারে রান, privileged, খোলা পোর্ট |	Dockle, docker-bench-security, Trivy config |
| DB/Redis/অন্য সার্ভিস ইমেজ |	mysql, redis, nginx	Trivy image scan |
