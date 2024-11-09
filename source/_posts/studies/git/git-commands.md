---
title: Git Commands
date: 2024-11-02 18:39:07
categories:
  - Studies
  - Git
##tags:
---
## git checkout

local repo에서 `HEAD`가 참조하는 commit이나 브랜치를 switch하기 위한 명령어입니다.

![Git Checkout](/images/git_checkout.png)

`git checkout`으로 현재 브랜치의 최근 commit과 `HEAD`가 불일치하면 “Detached HEAD” state가 되는데, 현재 개발작업과는 완전히 분리된 상태이므로 변경사항과 commit을 생성할 수 없습니다.

## git cherrypick

특정 브랜치의 commit을 체리처럼 따서 다른 브랜치의 commit으로 **copy + append**하여 변경사항을 적용하는 명령어입니다.

![Git Cherrypick](/images/git_cherrypick.png)

## git clone

remote repo를 복사하여 local repo를 생성할 때 사용하는 명령어로, 실행하면 다음과 같은 작업들이 수행됩니다.

- `main` branch에서 최근 commit에 해당하는 프로젝트 파일들이 로컬에 저장된다.
- remote history 전체를 로컬에 저장한다.
- 새로운 `main` branch를 생성한다.
- local repo 상에서 remote repo의 URL이 `origin`이라는 이름으로 등록된다.

## git commit --amend

현재 브랜치의 마지막 commit의 메시지 내용이나 적용할 변경사항들을 추가할 때 사용합니다.

amend를 수행한 뒤에는 `git push —-force`로 원격 브랜치와 동기화 시켜야 합니다.

## git config

git을 사용하면서 필요한 설정들을 scope별로 지정할 수 있는 명령어로, 가능한 scope들은 다음과 같습니다.

- local(=per local repo) : `${project_root}/.git/config`
- global(=per user) : `~/.gitconfig`
- system-wide(=per machine) : `$(prefix)/etc/gitconfig`

## git fetch vs. git pull

remote repo 단위 또는 origin 브랜치에 업데이트된 commit들의 정보를 확인하고 싶을 때는 `git fetch`, 추가된 commit들을 local 브랜치에 merge할 때는 `git pull`을 사용하면 됩니다.

여기서 두 명령어를 실행하기 전에는 반드시 올바른 local 브랜치로 checkout된 상태여야 합니다.

## git init

local repo를 생성하기 위한 명령어로, 실행하면 다음과 같은 작업들이 진행됩니다.

- cwd(current working directory)에 `.git`이라는 subDir가 생성된다.
- `main` branch가 생성된다.

## git merge

forked history를 다시 하나로 합치기 위한 명령어로, 보통 하나의 common base(공통 commit)에서 갈라진 branch들을 하나의 branch로 만들 때 사용합니다.

`git merge` + checkout + `git branch -d` 조합으로 많이 사용됩니다.

`git merge`를 하기 전에 준비과정이 필요합니다.

1. `git HEAD`(현재 commit)가 merge-receiving branch에 있는지 확인한다.
2. `git pull`로 remote repo 상의 merge-receiving branch의 최근 상태와 동기화한다.
3. `git merge`를 수행한다.

### merge하는 방법

#### Fast Foward

**FROM** branch에서 **TO** branch로 하나의 경로로 이어질 수 있을 때 사용할 수 있는 방법입니다.

![Fast Foward](/images/git_merge_ff.png)

Git은 branch들을 합치는 대신에 **FROM** branch tip이 **TO** branch tip으로 이동시켜서 하나의 history를 만들기에 새로운 merge commit을 생성하지 않습니다.

보통 `git rebase`와 함께 사용되어 사소한 feature나 bugfix를 할 때 사용합니다.

fast-foward 방식이 아닌 기록을 유지하기 위한 수단으로 merge commit을 생성하려면 `—-no-ff` 옵션을 사용하면 됩니다.

#### 3-way

Fast Forward merge와 다르게 **FROM** branch에서 feature를 개발하는 동안에 **TO** branch에서 계속 commit이 생성될 때도 사용할 수 있는 방식으로, 보통 규모가 큰 feature를 개발하거나 팀 단위로 project를 진행할 때 활용합니다.

![3-way](/images/git_merge_3_way.png)

3개의 commit 즉, **FROM** branch의 tip + **TO** branch의 tip + common base commit을 사용하는데, Git은 merge 전에 common base를 기준으로 두 tip에서 각각 어떤 부분이 달라졌는지를 확인합니다.

변한 부분이 겹치지 않는다면 두 변경사항들을 모두 반영한 새로운 merge commit을 생성하지만, 동일한 파일에서 겹치는 부분이 하나라도 있다면 두 변경사항 중 어떤 것을 반영할지 모르므로 Git은 merge conflict를 발생시켜 개발자의 개입을 요구합니다.

### Conflict

3-way merge를 할 때 발생하는 merge conflict는 보통 아래와 같은 상황에서 발생합니다.

- 두 사람이 동일한 파일에서 동일한 줄에 있는 코드를 수정한 경우
- 한 branch에서 잘 사용하던 파일을 다른 branch에서 삭제한 경우

현재 프로젝트의 working directory(unstage) 또는 stage 단계에서 대기하는 변경사항(pending changes)들로 commit을 만들지 않으면 Git은 merge를 시작하지 않습니다.

이에 대비하려면 `git stash`, `git checkout`, `git commit`, `git reset` 명령어로 local repo를 최근 상태로 만들면 됩니다.

merge 관련 유용한 command들은 아래와 같습니다.

- `git status` : confilct가 발생한 파일을 확인할 때 사용합니다.
- `git log -—merge` : merge되려는 두 branch에서 conflict가 발생한 commit들의 목록을 보여줍니다.
- `git diff` : repository와 파일들 사이의 차이점을 보여줍니다.

#### Resolving Conflict

merge 과정은 마치 만드려는 제품의 구성품 도안들을 겹치는 행위와 동일합니다. 2개 이상의 도안들이 같은 파트를 동시에 설계하여 중복된다면 conflict가 발생합니다.

![Mark1](/images/mark_1.jpeg)

#### feature → master(main)

평소에 `feature`에서 작업하는 중에 `master`에 새로운 commit이 올라올 때마다 master → feat 방향 merge를 수행합니다.

![Feat To Master](/images/feat_to_master.png)

#### feature → develop, test

`develop`, `test`는 각각 개발용, 검증용 환경에서 테스트하기 위한 브랜치로, 다른 분들에 의해서 live 환경에 추가할 기능들을 테스트하기 위한 목적으로 이미 많은 merge들이 이루어진 상태입니다.

`develop`, `test` → `feature` 방향으로 merge를 수행하여 해결해서는 **절대로 안됩니다.**

- 제 `feature`에 아직 배포준비가 되지 않은 다른 사람의 코드들이 반영되어 live에 배포될 수 있습니다.
- `feature`에서 작업하다가 다른 분이 생성한 코드를 수정하거나 삭제해서 다시 `develop`, `test`에 반영될 수 있습니다.

conflict는 아래와 같이 해결하면 됩니다.

1. `develop`, `test`로 checkout한 뒤에, 아래와 같은 브랜치를 생성한다.

```js
`${feat 이름}-${target 이름}`

// feature/FE2-123-dev
// feature/FE2-123-tst
```

2. `feature` → feat-target 방향으로 merge를 진행하면 당연히 충돌이 생길 것이다.

3. feat-target에 checkout된 상태에서 [VSC](https://code.visualstudio.com/docs/editor/versioncontrol##_merge-conflicts), [Intellij](https://javakong.tistory.com/217)에서 제공하는 자동화 툴로 충돌을 해결하고 3-way merge를 진행한다.

## git push

local repo의 history에 맞춰서 remote repo의 history에 commit들을 추가/삭제하는 명령어입니다.

`git push`를 할 때, remote 브랜치와 local 브랜치가 서로 갈라져서 non-fastfoward merge가 필요한 상황이면 git은 default로 이러한 push를 차단합니다.

이 상황에서는 2가지 해결방법이 있습니다.

- remote repo 브랜치에서 pull 받은 뒤, push 재수행
- `git push -—force` 로 강제로 history를 overwrite한다.

`git push -—force` 는 아래와 같은 상황에 필요합니다.

- `git reset` 이후
- `git commit -—amend` 이후
- `git rebase -i` 이후

git branch 이름 변경할 때도 활용할 수 있습니다.

```zsh
git push origin :<old-branch-name> <new-branch-name>
git push origin -u <new-branch-name>
```

## git rebase

`feature` 브랜치의 base를 특정 브랜치 또는 commit으로 옮겨서 현재까지 변경사항을 local에 반영하고 linear history를 유지할 때 사용하는 명령어로, rebase한 이후 `feature` 브랜치에는 새로운 commit들이 만들어집니다.

![Git Rebase](/images/git_rebase.png)

여기서 새로운 commit이 만들어지기 때문에 다른 팀원들과 공유하고 있는 브랜치를 로컬에서 rebase하고 push하면 **절대로 안됩니다.**


### interactive rebase

`feature` 브랜치를 rebase했을 때, 기존 commit들을 어떻게 재구성할지 commit 단위로 작업할 수 있습니다.

보통 commit 순서 수정, commit 메시지 수정, 이전 commit으로 squash, 제거 등 작업에 주로 활용됩니다.

`—-onto` 옵션을 붙이면 rebase되기 전후 브랜치를 명시하여 안전하게 변경사항들을 반영할 수 있습니다.

```zsh
git rebase --onto <newbase> <oldbase> <rebase-target-branch>
```

## git reset vs. git revert

간결하게 정리된 문서가 있어서 대신 공유드립니다😁

[git reset, revert로 이전 커밋으로 돌리기](https://kyounghwan01.github.io/blog/etc/git/git-reset-revert/##%E1%84%8B%E1%85%B5-%E1%84%8C%E1%85%A1%E1%86%A8%E1%84%8B%E1%85%A5%E1%86%B8%E1%84%8B%E1%85%B3%E1%86%AF-%E1%84%92%E1%85%A1%E1%84%82%E1%85%B3%E1%86%AB-%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%B2)

## git stash

staged + unstaged 변경사항들을 추후에 적용할 수 있도록 local repository의 stack에 stash 단위로 push/pop하는 명령어입니다.

local repo에서 작업하는 것이기 때문에 remote에는 영향이 가지 않고, `.gitignore`에 포함된 파일 또는 새롭게 생성된 unstaged 파일들은 포함하지 않습니다.

stack에 쌓인 stash는 WIP(Working In Progress)라는 id를 가지는데, stash id에 간단한 설명을 부여하는 것을 권장합니다.

가장 최근에 만들어진 stash가 `git stash` pop 0순위입니다.

stash를 생성할 때는 단일 파일, 다수 파일, 파일 일부를 단위로 할지 고를 수 있고, 가능한 stash 단위들을 "hunk"라고 합니다.

이미 stash로 추가된 변경사항들을 반영하는 브랜치도 생성할 수 있습니다.
