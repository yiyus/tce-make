#!/bin/sh
# tce-make: make tinycore extensions
#
# -- Jesus Galan (yiyus) 2015

USAGE='Usage: tce-make.sh [TCM]...'

if [ $# -gt 1 ]; then
	"$0" "$1" || exit 1
	shift
	echo
	exec "$0" "$@"
elif [ $# -eq 1 ]; then
	if [ "$1" = "-h" ]; then
		echo "$USAGE" 1>&2
		exit
	fi
	tcm="${1##*/}"
	name="${tcm%.tcm}"
	log="$name".
fi
exec 3>&2 >"$log"log 2>&1
echo3() { echo "$@" >&3; }
echo3 "$log"log

tce-load -i "squashfs-tools" || tce-load -wi "squashfs-tools" || exit 1

# $1 should be a tcm file
/usr/bin/awk -f - "$1" <<'EndOfAwk' | sh

function tcm_info(tcm) {
	if(split(tcm["Title:"], title) == 0) return
	info = title[2]".info"
	if(system("rm -f "info)) exit(1)
	for(i = 1; i <= length(fields); i++) {
		f = fields[i]
		txt = tcm[f":"]
		if(f == "Size") txt = sprintf("Size: %10sSIZE", "")
		else if(f == "Current") sub("^Current:[ \t]+", "&YYYY/mm/dd ", txt)
		print txt > info
		if(f == "Comments") {
			printf("%16s----\n", "") > info
			printf("%16sCompiled for Core X.x\n", "") > info
		}
	}
	n = split(tcm["Dependencies:"], dep)
	for(i = 2; n >= 2 && i <= n; i++) {
		print dep[i] > title[2]".dep"
	}
}

function tcm_make(tcm) {
	tcm_info(tcm)
	if(split(tcm["Git:"], git) == 2) {
		zip=git[2]
		sub(".*/", "", zip)
		print("wget "git[2]"/archive/master.zip -O "zip".zip")
		print("unzip "zip".zip")
	}
	n = split(tcm["Build-deps:"], dep)
	for(i = 2; n >= 2 && i <= n; i++) {
		print("( tce-load -i "dep[i]" || tce-load -wi "dep[i]" )")
	}
}

BEGIN{
	FIELDS="Title Description Version Author Original-site Copying-policy "\
		"Size Extension_by Tags Comments Change-log Current"
	split(FIELDS, fields)
}

(field == "Make:") {
	print
	next
}

/^[ \t]*(#.*)+$/ { next }

/^[A-Z][a-z_-]+:/ {
	field = $1
	if(field == "Title:") {
		tcm_info(tcm)
	} else if(field == "Make:") {
		tcm_make(tcm)
		sub("Make:[ \t]*", "")
		if($0 != "") print
		next
	}
	tcm[field] = $0
	next
}

/^[ \t]/ && (field != "") { tcm[field] = tcm[field]"\n"$0 }

END { print "exit" }

EndOfAwk

for info in "$name"*.info; do
	dir="${info%.tcz.info}"
	tcz="$dir".tcz

	# make tcz
	if [ ! -d "$dir" ]; then
		echo3 "ERROR: $dir/ not found. Cannot create $tcz"
		exit 1
	fi
	rm -f "$TCZ".tcz
	mksquashfs "$dir" "$tcz" -all-root && echo3 "$tcz"

	# md5, list and info
	md5sum "$tcz" > "$tcz".md5.txt && echo3 "$tcz".md5.txt
	(cd "$dir"; find * ! -type d) > "$tcz".list && echo3 "$tcz".list
	sed '/^Size:/s,SIZE,'$(du "$tcz" | cut -f1)'K,
		/Compiled for Core X.x/s/X/'$(version | cut -d. -f1)'/
		/^Current:/s,YYYY/mm/dd,'$(date +%Y/%m/%d)',
	' -i "$info" && echo3 "$info"
	
	# pack to .tar.gz.bfe
	tgz="$dir".tar.gz
	bfe="$tgz".bfe
	FILES="$tcz $tcz.info $tcz.list $tcz.md5.txt"
	for f in $FILES; do
		[ -f "$f" ] && continue
		echo3 "ERROR: $f not found. Cannot create $bfe"
		exit 1
	done
	dep=''
	if [ -f "$tcz".dep ]; then
		dep="$tcz".dep
		echo3 "$tcz".dep
	fi
	tar -zcvf "$tgz" $FILES ${dep:+"$dep"} && echo3 "$tgz"
	echo -e 'tinycore\ntinycore\n' | bcrypt "$tgz" && echo3 "$tgz".bfe
done
