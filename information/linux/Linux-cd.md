# cd
 /etc/profile
alias cd1="cd .."
alias cd2="cd ../.."
alias cd3="cd ../../.."
alias cd4="cd ../../../.."
alias cd5="cd ../../../../.."
alias cd6="cd ../../../../../.."

function mkdircd () { 
    mkdir -p "$@" && eval cd "\"\$$#\"";
}

cd - �����ļ����л�


shopt -s cdspell ����ƴд����


# grep
grep John /etc/passwd ��ʾƥ���
grep -v John /etc/passwd ��ʾ��ƥ���
grep -c John /etc/passwd ������ƥ��
grep -cv John /etc/passwd �����в�ƥ��
grep -i john /etc/passwd �����ִ�Сдignore
grep -r john /home/users ��Ŀ¼����

# find
find /etc -name "*mail*"
find / -type f -size +100M ��ʾ����100M���ļ�

# join 
join employee.txt bonus.txt

tr a-z A-Z < employee.txt ���ĵ��е�Сдȫ��ת��Ϊ��д

stat /etc/my.cnf ��ʾ�ļ����ļ�����Ϣ

stat -f / ��ʾ�ļ�ϵͳ״̬

# zip
zip var-log-files.zip /var/log/*

unzip var-log.zip

tar xvfz /tmp/my_home_directory.tar.gz ��ѹ


ctrl+R����command��ʷ

dd if=/dev/zero of=/home/swap-fs bs=1M count=512 ����swap�ļ�


df �Ch

sar �Cu

vmstat 1 100
 