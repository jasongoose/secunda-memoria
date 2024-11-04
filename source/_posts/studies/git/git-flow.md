---
title: Git Flow
date: 2024-11-02 18:38:59
categories:
  - Studies
  - Git
#tags:
---

Git을 사용하는 development model들 중 하나로, 역할이 정해진 다양한 branch들을 사용합니다.

## Main Branches

![Main Branches](/images/main_branches.png)

### master(main)

`origin/master` 브랜치의 `HEAD`는 production-ready state를 가집니다.

`master` 브랜치에 merge가 일어날 때마다 build를 거쳐서 새로운 production-release가 만들어집니다.

### develop

`origin/develop` 브랜치의 `HEAD`는 추후에 있을 release를 위해 이루어진 최신 개발변화를 반영합니다.

release될 준비를 마치면 master 브랜치에 merge되면서 release number(version number)가 적힌 tag가 달립니다.

## Support Branches

main 브랜치들을 기능적으로 지원하기 위한 브랜치들로, 다른 브랜치의 특정 commit에서 생성되면 시간이 지난 뒤에 다른 브랜치로 merge됩니다.

### feature(topic)

예정된 release를 위해서 새로운 feature들을 개발할 때 `develop` 브랜치의 `HEAD`에서 생성하는 브랜치입니다.

feature 개발이 완료되면 다시 `develop` 브랜치로 merge하고, 추후 히스토리로 참고하기 위해 보통 삭제하지 않습니다.

![Feature](/images/feature_merge.png)

### release

새로운 production-release(= `origin/master` 브랜치로 merge) 직전에 대기하는 브랜치입니다.

![Release](/images/release.png)

계획했던 release 방안과 거의 가깝게 개발되었을 때, `develop` 브랜치 `HEAD`로부터 새로운 version number를 부여받은 `release` branch를 생성합니다.

minor bugfix를 진행하거나 release를 위한 meta-data를 준비하는 동안 `develop` 브랜치는 다음 release를 위한 `feature`들을 계속 수용하고, 현재 `release`에서 bugfix를 할 때마다 다음 release 개발과정에 반영될 수 있도록 `develop` 브랜치에 merge합니다.

release될 준비를 마치면 `master` 브랜치로 merge한 뒤에 보통 삭제됩니다.

### hotfix

운영 중에 빠른 해결이 필요한 에러가 발생할 때마다 `master` 브랜치 `HEAD`에서 생성하는 브랜치로, 새로운 version number를 부여받습니다.

![Hotfix](/images/hotfix.png)

commit하면서 trouble shooting을 진행하고 완료되면 다음 release에도 똑같은 문제가 발생하지 않도록 `develop` 브랜치에 merge한 뒤에 `master` 브랜치로 merge합니다.

## Terms

### Pull Request

개발자(contributor)가 작업했던 source repo의 branch를 destination repo의 branch로 merge 시키기 전에 repo 담당자(maintainer)를 포함한 다른 개발자들이 전과 후의 코드의 diff를 바탕으로 피드백을 얻는 메커니즘입니다.

PR을 거치면서 개선해야할 점들이 댓글로 올라오면 source branch에서 다시 commit들을 생성하여 수정한 다음에 피드백을 다시 받을 수 있습니다.

정해진 횟수 이상의 승인을 받은 뒤에 maintainer는 source branch를 destination branch로 merge시킨 다음에 해당 pr을 닫습니다.

### Staging Area

Git의 3가지 상태들 중 하나로 working directory(wd)와 remote repo history 사이의 buffer 역할을 하는 영역입니다.

staging area을 활용하면 wd 내에서 발생한 모든 pending change들 중, 관련된 건들만 논리적 단위로 모아서 단일 commit을 생성할 수 있습니다.

commit의 크기는 추후 revert와 디버깅을 위해서 atomic하게 만드는 것을 권장합니다.
