Assignment 1: Pushy

主要是用shell实现git的功能，难度较高

支持作业、考试辅导、代写，具体可以加微信lxhao580，老师直接接单，不经过中介平台，价格优惠，服务靠谱

部分代码：
```
root@c0e23c71e7d3:~# cat pigs-rm
#!/bin/dash
# check if the repository directory exists, and create the index directory and deleted files list if they do not exist
check_init() {
  if [ ! -d .pig ]; then
      echo "pigs-add: error: pigs repository directory .pig not found"
      exit 0
  fi
  if [ ! -d .pig/index ]; then
    mkdir .pig/index/
  fi
  if [ ! -f .pig/index/.deleted ]; then
    touch .pig/index/.deleted
  fi
}

check_init

force=0
cached=0
filenames=""

# parse command-line arguments
for arg in "$@"; do
    case $arg in
        --force)
            force=1
            ;;
        --cached)
            cached=1
            ;;
        *)
            filenames="${filenames} ${arg}"
            ;;
    esac
done

# get the commit ID of the latest commit
commit_id=$(find .pig/commit/ -mindepth 1 -maxdepth 1 -type d 2>/dev/null | wc -l)
commit_id=$((commit_id - 1))

for filename in $filenames; do
    if [ $force -eq 0 ] && [ $cached -eq 1 ];then
      if [ -f .pig/index/"$filename" ] ; then
        if ! cmp -s .pig/index/"$filename" .pig/commit/$commit_id/"$filename"; then
          if ! cmp -s .pig/index/"$filename" "$filename"; then
              echo "pigs-rm: error: '$filename' in index is different to both the working file and the repository"
              exit 1
          fi
        fi

        if ! cmp -s .pig/index/"$filename" "$filename"; then
          echo "pigs-rm: error: '$filename' has staged changes in the index"
          exit 1
        fi
      fi
    else
      # compare the file with the version in the latest commit
      if [ $force -eq 0 ] && ! cmp -s "$filename" .pig/commit/$commit_id/"$filename" && cmp -s "$filename" .pig/index/"$filename"; then
        echo "pigs-rm: error: '$filename' has staged changes in the index"
        exit 1
      fi
      # deleted in index
      if [ $force -eq 0 ] && [ -f "$filename" ] && [ -f .pig/index/"$filename" ] && grep -q -x "$filename" .pig/commit/"$commit_id"/.deleted 2>/dev/null ; then
        echo "pigs-rm: error: '$filename' has staged changes in the index"
        exit 1
      fi
      # compare the file with the version in the latest commit
      if [ $force -eq 0 ] && [ -f "$filename" ] && [ -f .pig/commit/$commit_id/"$filename" ] && ! cmp -s "$filename" .pig/commit/$commit_id/"$filename"; then
        if [ -f .pig/index/"$filename" ] && ! cmp -s .pig/index/"$filename" "$filename"; then
            echo "pigs-rm: error: '$filename' in index is different to both the working file and the repository"
            exit 1
        fi
        echo "pigs-rm: error: '$filename' in the repository is different to the working file"
        exit 1
      fi
      # compare the file with the version in the index
      if [ $force -eq 0 ] && [ -f "$filename" ] && [ -f .pig/index/"$filename" ] && ! cmp -s "$filename"  .pig/index/"$filename" ; then
        echo "pigs-rm: error: '$filename' has staged changes in the index"
        exit 1
      fi
    fi

    if [ ! -f .pig/index/"$filename" ] && grep -q -x "$filename" .pig/commit/"$commit_id"/.deleted 2>/dev/null; then
      echo "pigs-rm: error: '$filename' is not in the pigs repository"
      exit 1
    fi
    if [ ! -f .pig/index/"$filename" ] && grep -q -x "$filename" .pig/index/.deleted 2>/dev/null; then
      echo "pigs-rm: error: '$filename' is not in the pigs repository"
      exit 1
    fi
    # file has not been indexed, output error message and skip
    if [ ! -f "$filename" ] &&  [ ! -f .pig/commit/$commit_id/"$filename" ]; then
      echo "pigs-rm: error: '$filename' is not in the pigs repository"
      exit 1
    fi
    if [ ! -f .pig/index/"$filename" ] && [ ! -f .pig/commit/"$commit_id"/"$filename" ]; then
      echo "pigs-rm: error: '$filename' is not in the pigs repository"
      exit 1
    fi
    # file has been indexed, but has already been deleted in the working directory
    if [ $force -eq 0 ] && [ ! -f "$filename" ]; then
        echo "pigs-rm: error: '$filename' has staged changes in the index"
        exit 1
    fi
    # remove the file from the index
    if [ -f .pig/index/"$filename" ]; then
        rm .pig/index/"$filename"
    fi
    # add the file name to the deleted files list in the index
    echo "$filename" >> .pig/index/.deleted

    # if --cached option is not used, remove the file from the working directory
    if [ $cached -eq 0 ]; then
        rm "$filename"
    fi
done
```

