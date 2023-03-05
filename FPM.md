1、查看php-fpm的进程个数
ps -ef |grep "php-fpm"|grep "pool"|wc -l


2、查看每个php-fpm占用的内存大小
ps -ylC php-fpm --sort:rss


3.查看PHP-FPM在你的机器上的平均内存占用
ps --no-headers -o "rss,cmd" -C php-fpm | awk '{ sum+=$1 } END { printf ("%d%s\n", sum/NR/1024,"M") }'


4.查看单个php-fpm进程消耗内存的明细
pmap $(pgrep php-fpm) | less


php-fpm的参数优化
pm = dynamic #对于专用服务器，pm可以设置为static。#如何控制子进程，选项有static和dynamic。如果选择static，则由pm.max_children指定固定的子进程数。如果选择dynamic，则由下开参数决定：
pm.max_children #子进程最大数
pm.start_servers #启动时的进程数
pm.min_spare_servers #保证空闲进程数最小值，如果空闲进程小于此值，则创建新的子进程
pm.max_spare_servers #保证空闲进程数最大值，如果空闲进程大于此值，此进行清理


使用 pm static 优化你的服务器性能
PHP-FPM 的 static 设置取决于你服务器有多少闲置内存。大多数情况下，如果你服务器的内存不足，那么 PM 设置成 ondemand 或 dynamic 将是更好的选择。但是，一旦你有可用的闲置内存，那么把 PM 设置成 static 的最大值将减少许多 PHP 进程管理器（PM）所带来的开销。换句话说，你应该在没有内存不足和缓存压力的情况下使用 pm.static 来设置 PHP-FPM 进程的最大数量。此外，也不能影响到 CUP 的使用和其他待处理的 PHP-FPM 操作。