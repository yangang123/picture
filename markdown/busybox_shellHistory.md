# 说明
busybox 1.20.2中的shell的实现历史命令的保存，整个命令都是保存在

## 启动shell加载history命令
```c
static void load_history(line_input_t *st_parm)
fp = fopen_for_read(st_parm->hist_file);
if (fp) {
        -> st_parm->history[i++] = line; //拷贝所有的history到命令
```
## 关闭shell,进行保存命令
```c
static void load_history(line_input_t *st_parm)
   fp = xfdopen_for_write(fd);
    for (i = 0; i < st_temp->cnt_history; i++)
    fprintf(fp, "%s\n", st_temp->history[i]);
```
## 保存commande到history内存中
```c	
    if (command_len > 0)
		remember_in_history(command); 
            state->history[i++] = xstrdup(str);  //保存commmander
            state->cur_history = i;
            state->cnt_history = i;
            save_history(str); //保存到文件中
```

# 命令的数据结构
从下面可以看出history是1个数组保存所有的history
```c
typedef struct line_input_t {
	int flags;
	const char *path_lookup;
# if MAX_HISTORY
	int cnt_history;
	int cur_history;
	int max_history; /* must never be <= 0 */
#  if ENABLE_FEATURE_EDITING_SAVEHISTORY
	/* meaning of this field depends on FEATURE_EDITING_SAVE_ON_EXIT:
	 * if !FEATURE_EDITING_SAVE_ON_EXIT: "how many lines are
	 * in on-disk history"
	 * if FEATURE_EDITING_SAVE_ON_EXIT: "how many in-memory lines are
	 * also in on-disk history (and thus need to be skipped on save)"
	 */
	unsigned cnt_history_in_file;
	const char *hist_file;
#  endif
	char *history[MAX_HISTORY + 1];
# endif
} line_input_t;

```