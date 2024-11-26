```sh
#/bin/bash  
  
has_a=$(echo "$@" | grep -q "\-a"; echo $?)  
current_branch=$(git branch --show-current)  
changes=$(git status --porcelain)  
has_changes=$(( ${#changes} > 0 ? 1 : 0 ))  
body_sample=$(cat "$HOME/.git-body-sample")  
  
echo "has_changes: $has_changes"  
  
if [ ! $has_changes -eq 0 ]; then  
	echo "You have changes. it is staged now."  
	git add .  
fi  
  
read -rep "Title: " title  
read -rep "Base branch (ex develop): " base_branch  
  
if [ ! -n "$base_branch" ]; then  
        base_branch="develop"  
fi  
  
git commit -m "$title"  
git push -u origin $currnet_branch  
  
echo "current_branch: $current_branch"  
echo "base_branch: $base_branch"  
  
pr_number=$(gh pr list --state open --base "$base_branch" --head "$current_branch" --json number -q '.[0].number')  
  
if [ ! -n "$pr_number" ]; then  
		gh pr create --title "$title" --body "$body_sample" --assignee "@me" --base "$base_branch" --fill  
fi  
  
if [ ! "$has_a" -eq 0 ]; then  
read -rep "Merge to alpha(y/n)? " merge_alpha_yn  
	if [ "$merge_alpha_yn" = "n" ]; then  
	exit 0  
	fi  
fi  
  
git checkout alpha  
git pull origin alpha  
git merge --no-ff --no-edit $current_branch  
git push origin alpha  
git checkout $current_branch
```