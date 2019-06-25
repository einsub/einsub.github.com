---
layout: post
title: "지속적 통합/배포(CI/CD)를 위한 Git Workflow 전략"
date: 2019-06-25 14:34:15
author: Reid
categories:
  - engineering
tags:
  - continuous integration
  - continuous distribution
  - ci/cd
  - git
  - workflow
  - branch strategy
  - rebase
published: true
---

## Git Flow

2010년 Vicent Driessen이라는 분이 만든 가장 널리 알려진 Git 작업 절차입니다. Git Flow는 `master`와 `develop`이라는 항상 존재하는 주 브랜치가 있고, `feature-*`, `hotfix-*`, `release-*`라는 필요에 따라 생성하는 브렌치가 있습니다. 물론, 이후 `improvement-*`, `bugfix-` 등 프로젝트에 따라 다양한 브랜치 모델이 추가되기도 하였습니다.

이 절차는 다음과 같은 형태로 진행됩니다.

1. `master` 브랜치에서 `develop` 브랜치를 분기합니다.
2. 개발자들은 `develop` 브랜치에 자유롭게 커밋을 합니다.
3. 기능 구현이 있는 경우 `develop` 브랜치에서 `feature-*` 브랜치를 분기합니다.
4. 배포를 준비하기 위해 `develop` 브랜치에서 `release-*` 브랜치를 분기합니다.
5. 테스트를 진행하면서 발생하는 버그 수정은 `release-*` 브랜치에 직접 반영합니다.
6. 테스트가 완료되면 `release` 브랜치를 `master`와 `develop`에 merge합니다.

그리고 이 절차가 반복됩니다.


프로덕트를 빌드하는 일이 힘들고 시간이 걸리던 과거가 있었습니다. 개발자들은 빌드를 위해 수동으로 컴포넌트들을 통합하고 빌드했으며, 그렇게 어렵게 만들어진 결과물은 테스트를 위해 또 많은 시간을 보내야 했습니다. Git Flow가 인기를 얻었던 것은 이러한 그 시대의 개발 인프라에 만족했기 때문입니다.

이 flow는 배포 시기가 명확한 프로젝트에 특화되어 있습니다. 배포가 없을 때에는 개발자가 자유롭게 커밋을 진행하지만, 배포를 위해 `release-*` 브랜치가 만들어지면 집중적으로 버그 수정을 합니다. 이는 전통적인 방식의 개발 흐름에서는 문제가 없지만 지속적 통합/배포가 가능해진 요즘에는 오히려 문제를 일으킵니다. 배포 가능한 버전을 만들기 위한 사전 작업이 필요하기 때문이죠. 배포를 하려면 각 잡고 몰아치기를 해야 합니다.

최근에는 프로덕트를 통합하고 배포하는 일이 클라우드에서 자동으로 이루어집니다. 커밋만 올리면 알아서 동작하므로, 누가 되었든 항상 최신 빌드를 사용 할 수 있습니다. 이런 개발 프로세스의 발전으로 인해 이제 더 이상 예전처럼 배포 주간을 두고 배포를 위한 브랜치를 만들어 따로 관리 할 필요가 없습니다.

## GitHub Flow

2011년 GitHub에서 제시한 워크플로는 언제나 배포 가능한 빌드를 만들기 위한 방안이었습니다. Git Flow가 release를 위한 브랜치를 따로 두고 집중적으로 테스트를 하는 것과 달리 GitHub Flow는 커밋이나 `feature-*` 브랜치에서 테스트를 완료하고 Pull Request와 Code Review를 거쳐 `master` 브랜치에 merge합니다. 따라서, `release-*` 브랜치 없이도 항상 배포 가능한 상태를 유지 할 수 있게 됩니다. 반면에 자율적인 테스트에 의존하기 때문에 배포에 앞서서 철저한 테스트 과정을 거치는 Git Flow에 비해 빌드의 안정성이 떨어질 가능성이 있습니다.

## 이 둘을 통합 할수는 없을까?

GitHub Flow처럼 개발자들이 지속적으로 브랜치를 merge 하지만, 통합 테스트가 완료된 커밋들만 선별적으로 배포를 할 수는 없을까요? GitHub Flow에 staging 브랜치를 추가하고 interactive rebase를 이용해 develop 브랜치로부터 원하는 커밋들만 골라서 가져올 수 있습니다.

이 방법을 이용하려면 `develop`, `staging`, `production`, 이렇게 세 가지의 상시 브랜치를 유지해야 합니다. `develop`은 Pull Request를 거친 커밋들이 자유롭게 올라갑니다. 배포 보다는 기능 구현에 초점을 맞춘 브랜치이므로, 배포 주기에 구애받지 않고 커밋을 올릴 수 있습니다. `staging` 브랜치는 배포가 확정 된 커밋들이 있을 곳입니다. 참고로 이 `staging` 빌드는 Git Flow의 `release` 브랜치와 성격이 다릅니다. `release`가 배포를 위해 브랜치를 분기시켜 테스트와 버그 수정을 한 후 배포가 완료되면 다시 원래의 브랜치로 merge되는 반면, `staging` 브랜치는 지속적으로 유지됩니다. 따라서, 지속적 통합/배포 기능을 구축해두었다면 팀원 누구든 배포 가능한 빌드를 staging 빌드로 테스트 해 볼 수 있습니다.

다시 한번 정리해 보겠습니다.

|브랜치|설명|
---|---
|develop|개발자들이 PR을 통해 커밋을 최초로 등록하는 브랜치입니다. 코드에 대한 리뷰를 거쳤다 하더라도 3rd party 플랫폼이나 서버/클라이언트간의 통합 테스트는 덜 된 커밋들이 존재 할 수도 있습니다. production 배포와 관계가 없으므로 개발자들은 큰 부담 없이 커밋을 올릴 수 있습니다.|
|staging|develop 환경의 테스트를 마치고 배포가 결정 된 커밋들을 옮깁니다. 이 브랜치의 빌드는 production과 동일한 환경에서 테스트 되므로 순수하게 배포되는 기능 혹은 버그 수정에 집중 할 수 있습니다. 개발 환경에서의 불완전한 설정이나 비정상적인 데이터가 아닌 실제 프로덕션 환경에서의 테스트는 새로운 버그나 개선점을 찾아 낼 수 있습니다.
|production|`staging` 브랜치에서 테스트가 완료되면, `production` 브랜치를 `staging` 브랜치로 merge하는 것만으로 배포가 완료됩니다.|

## Rebase를 활용한 Git Flow

예제를 통해 개발자의 커밋들이 `develop` 브랜치를 거쳐 `staging`으로 merge 되고, 이후 `production`으로 배포되는 과정까지 살펴보겠습니다.

### develop 브랜치

```text
* 0d9dd2e - Update test.txt again - Reid (HEAD -> feature/featureA)
* 6f72417 - Update test.txt - Reid
* cb0ab0e - Add test.txt - Reid (develop, origin/develop, staging, origin/staging, production, origin/production)
```

처음에는 당연히 `develop`, `staging`, `production` 브랜치가 하나의 커밋에서 시작 될 겁니다. `feature/featureA`라는 topic 브랜치를 만들어 3개의 커밋으로 개발을 완료했습니다. 이제 `develop` 브랜치로 올리기 위해 Pull Request를 요청해야 합니다. 보통은 PR을 올린 후 리뷰를 거쳐 승인이 되면 GitHub이나 Bitbucket에서 제공하는 버튼을 이용해 수월하게 merge를 진행하겠지만, 일단 우리는 Git CLI로 직접 해보겠습니다.

```sh
$ git checkout develop
$ git merge --squash feature/featureA
Updating cb0ab0e..0d9dd2e
Fast-forward
Squash commit -- not updating HEAD
 test.txt | 3 ++-
 1 file changed, 2 insertions(+), 1 deletion(-)
$ git commit -m 'Merged in bugfix/remove-unused-code (pull request #1)’
$ git push origin develop
```

`develop` 브랜치에서 `feature/featureA` 브랜치를 merge 합니다. 이 때 `--squash` 옵션을 주어 여러 커밋을 하나의 커밋으로 모아서 develop 브랜치에 새로 커밋을 만들어줍니다. 이렇게 하는 이유는 `develop` 브랜치에 한 feature 당 하나의 커밋, 한 bugfix 당 하나의 커밋을 유지하여 이후 `staging` 브랜치에 선택적으로 커밋을 옮길 때 수월하게 구분 할 수 있게 하기 위해서입니다.

```text
* fbeca95 - Merged in bugfix/remove-unused-code (pull request #1) - Reid (HEAD -> develop, origin/develop)
| * 0d9dd2e - Update test.txt again - Reid (feature/featureA)
| * 6f72417 - Update test.txt - Reid
|/
* cb0ab0e - Add test.txt - Reid (staging, origin/staging, production, origin/production)
```

squash merge가 완료된 모습입니다. 필요에 따라 이후에 커밋 히스토리를 참조하거나 재수정을 위해 기존의 `feature/featureA` 브랜치를 남겨둘 수도 있습니다만 여기서는 깔끔한 히스토리를 위해 삭제하겠습니다.

```sh
$ git branch -D feature/featureA
Deleted branch feature/featureA (was 0d9dd2e).
```

지금까지의 모습은 이렇습니다.

```text
* fbeca95 - Merged in bugfix/remove-unused-code (pull request #1) - Reid (HEAD -> develop, origin/develop)
* cb0ab0e - Add test.txt - Reid (staging, origin/staging, production, origin/production)
```

이런 식으로 커밋을 몇개 더 만들어 보겠습니다.

```text
* 83174e6 - Merged in feature/fix-char-order (pull request #5) - Reid (HEAD -> develop, origin/develop)
* 816108a - Merged in bugfix/add-missing-init-code (pull request #4) - Reid
* 02f3054 - Merged in feature/add-new-line (pull request #3) - Reid
* db3cbf3 - Merged in bugfix/fix-typo (pull request #2) - Reid
* fbeca95 - Merged in bugfix/remove-unused-code (pull request #1) - Reid
* cb0ab0e - Add test.txt - Reid (staging, origin/staging, production, origin/production)
```

### staging 브랜치

최근의 개발 환경은 third party 플랫폼이나 다양한 외부 서비스와의 연계로 인해 통합 테스트가 반드시 필요합니다. 하지만 개발자가 로컬 환경에서 완벽하게 통합 테스트를 진행하는 것은 쉬운 일이 아닙니다. 그러므로 개발자들의 커밋은 완벽한 테스트를 거치지 않았더라도 스스로 납득할만한 수준에서 테스트를 진행하고 코드 리뷰를 거쳐 `develop 브랜치`에 올라갈 수 있어야 합니다.

개발은 배포로 인해 멈추어서는 안됩니다. `develop` 브랜치는 배포와 관계없이 지속적으로 PR이 올라오고 merge가 됩니다. 배포가 되기 위해 커밋들이 취합되는 브랜치는 바로 `staging` 브랜치입니다.

`develop` 브랜치의 pull request #1 커밋과 pull request #3 커밋이 배포 준비가 되었습니다. 이 두 커밋을 `staging` 브랜치로 interactive rebase를 진행하겠습니다. 먼저 `staging` 브랜치를 `develop` 브랜치로 merge해서 HEAD를 최신 커밋으로 맞춰준 후에 `origin/staging`으로 interactive rebase를 시킵니다. 이 경우에는 merge 대신 `git reset develop --hard` 명령도 동일한 결과를 냅니다.

```sh
$ git checkout staging
$ git merge develop
$ git rebase -i origin/staging
```

Interactive rebase는 처음 접하면 복잡해보이고 실수를 할까봐 두려울 수 있습니다. 하지만, 별로 걱정 할 필요가 없습니다. rebase를 하는 과정은 remote로 push만 하지 않으면 모든 변경 사항이 로컬에만 반영되기 때문에 중간에 문제가 생겼을 때 `rebase --abort`로 취소한다거나 `git reset --hard` 명령으로 원상 복구를 하면 됩니다. interactive rebase는 브랜치에 존재하는 여러 커밋들을 하나로 합치거나, 순서를 바꾸거나, 커밋 메시지를 바꾸거나, 원하는 커밋만 선택해서 rebase를 할 수 있습니다.

`rebase -i` 명령으로 interactive rebase를 하면 에디터가 실행되고 다음과 같이 커밋 목록이 나타납니다.

```text
  1 pick fbeca95 Merged in bugfix/remove-unused-code (pull request #1)
  2 pick db3cbf3 Merged in bugfix/fix-typo (pull request #2)
  3 pick 02f3054 Merged in feature/add-new-line (pull request #3)
  4 pick 816108a Merged in bugfix/add-missing-init-code (pull request #4)
  5 pick 83174e6 Merged in feature/fix-char-order (pull request #5)
  6
  7 # Rebase 7a9b341..7611684 onto 7a9b341 (5 commands)
  8 #
  9 # Commands:
 10 # p, pick <commit> = use commit
 11 # r, reword <commit> = use commit, but edit the commit message
 12 # e, edit <commit> = use commit, but stop for amending
 13 # s, squash <commit> = use commit, but meld into previous commit
 14 # f, fixup <commit> = like "squash", but discard this commit's log message
 15 # x, exec <command> = run command (the rest of the line) using shell
 16 # d, drop <commit> = remove commit
 17 # l, label <label> = label current HEAD with a name
 18 # t, reset <label> = reset HEAD to a label
 19 # m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
 20 # . create a merge commit using the original merge commit’s
 21 # . message (or the oneline, if no original merge commit was
 22 # . specified). Use -c <commit> to reword the commit message.
 23 #
 24 # These lines can be re-ordered; they are executed from top to bottom.
 25 #
 26 # If you remove a line here THAT COMMIT WILL BE LOST.
 27 #
 28 # However, if you remove everything, the rebase will be aborted.
 29 #
 30 #
 31 # Note that empty commits are commented out
```

우리는 #1, #3 PR만을 선택 할 것이므로 내용을 다음과 같이 수정하고 저장합니다. 예전 커밋이 위로 올라오는 오름차순이어서 평소 내림차순 커밋 목록에 익숙한 분들께 헷갈릴 수 있으니 주의해야 합니다.

```text
pick fbeca95 Merged in bugfix/remove-unused-code (pull request #1)
drop db3cbf3 Merged in bugfix/fix-typo (pull request #2)
pick 02f3054 Merged in feature/add-new-line (pull request #3)
drop 816108a Merged in bugfix/add-missing-init-code (pull request #4)
drop 83174e6 Merged in feature/fix-char-order (pull request #5)
```

#1과 #3 커밋을 제외한 나머지 커밋들의 명령어를 `pick`에서 `drop`으로 변경했습니다. `drop`으로 변경하는 것도 귀찮다면 그냥 해당 라인을 지워버려도 동일하게 동작합니다.

```text
pick fbeca95 Merged in bugfix/remove-unused-code (pull request #1)
pick 02f3054 Merged in feature/add-new-line (pull request #3)
```

이제 에디터에서 저장을 하고 나면 커밋 그래프가 다음처럼 변경되어 있는 것을 볼 수 있습니다.

```text
* e27bf0c - Merged in feature/add-new-line (pull request #253) - Reid (HEAD -> staging)
| * 83174e6 - Merged in feature/fix-char-order (pull request #257) - Reid (origin/develop, develop)
| * 816108a - Merged in bugfix/add-missing-init-code (pull request #254) - Reid
| * 02f3054 - Merged in feature/add-new-line (pull request #253) - Reid
| * db3cbf3 - Merged in bugfix/fix-typo (pull request #251) - Reid
|/
* fbeca95 - Merged in bugfix/remove-unused-code (pull request #1) - Reid
* 7a9b341 - Add test.txt - Reid (production, origin/staging, origin/production)
```

PR #1은 어차피 `develop`과 `staging`의 공통 조상 커밋이기 때문에 그로부터 분기가 되는 것을 볼 수 있습니다. 이 상황에서 `staging` 브랜치를 `origin/staging`으로 push를 합니다. 만약, 지속적 배포가 동작한다면 알아서 staging 빌드가 완성되어 테스트 가능한 상황이 만들어 질 것입니다.

다른 명령어를 이용하기 위해 한번만 더 interactive rebase를 진행해보겠습니다.

```sh
$ git push origin staging
$ git reset develop —hard
$ git rebase -i origin/staging
```

`staging` 브랜치를 `develop` 브랜치로 강제 이동시키고 다시 interactive rebase를 실행합니다.

```text
  1 pick db3cbf3 Merged in bugfix/fix-typo (pull request #2)
  2 pick 816108a Merged in bugfix/add-missing-init-code (pull request #4)
  3 pick 83174e6 Merged in feature/fix-char-order (pull request #5)
  4
  5 # Rebase e27bf0c..83174e6 onto e27bf0c (3 commands)
  6 #
  7 # Commands:
  8 # p, pick <commit> = use commit
  9 # r, reword <commit> = use commit, but edit the commit message
 10 # e, edit <commit> = use commit, but stop for amending
 11 # s, squash <commit> = use commit, but meld into previous commit
 12 # f, fixup <commit> = like "squash", but discard this commit's log message
 13 # x, exec <command> = run command (the rest of the line) using shell
 14 # d, drop <commit> = remove commit
 15 # l, label <label> = label current HEAD with a name
 16 # t, reset <label> = reset HEAD to a label
 17 # m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
 18 # . create a merge commit using the original merge commit’s
 19 # . message (or the oneline, if no original merge commit was
 20 # . specified). Use -c <commit> to reword the commit message.
 21 #
 22 # These lines can be re-ordered; they are executed from top to bottom.
 23 #
 24 # If you remove a line here THAT COMMIT WILL BE LOST.
 25 #
 26 # However, if you remove everything, the rebase will be aborted.
 27 #
 28 #
 29 # Note that empty commits are commented out
```

`develop` 브랜치와 `origin/staging` 브랜치 사이에서 분기된 커밋이 4개인데, 왜 3개밖에 표시되지 않는 것일까요? 그것은 바로 첫 번째 rebase 작업에서 이미 하나의 커밋이 복사가 되었기 때문에 중복 커밋을 제외시킨 것입입니다. 이번 rebase 작업에서는 #2 PR과 #4 PR을 하나의 커밋으로 합쳐서 staging으로 옮기려고 합니다.

```text
reword db3cbf3 Merged in bugfix/fix-typo (pull request #2)
fixup 816108a Merged in bugfix/add-missing-init-code (pull request #4)
drop 83174e6 Merged in feature/fix-char-order (pull request #5)
```

#2 PR은 `reword`로 명시해서 커밋을 사용하지만 메시지는 다시 작성할 것이고, #4 PR은 #2 커밋과 squash 시키면서 로그 메시지는 포기 할 것입니다. (유지하려면 squash 명령어를 쓰면 됩니다) 이 상태에서 저장을 하면 다음과 같이 새로운 커밋 메시지를 입력 할 수 있게 해줍니다.

```text
  1 Merged in bugfix/fix-typo (pull request #2)
  2
  3 # Please enter the commit message for your changes. Lines starting
  4 # with '#' will be ignored, and an empty message aborts the commit.
  5 #
  6 # interactive rebase in progress; onto e27bf0c
  7 # Last command done (1 command done):
  8 # reword db3cbf3 Merged in bugfix/fix-typo (pull request #2)
  9 # Next commands to do (2 remaining commands):
 10 # fixup 816108a Merged in bugfix/add-missing-init-code (pull request #4)
 11 # drop 83174e6 Merged in feature/fix-char-order (pull request #5)
 12 # You are currently rebasing branch 'staging' on ‘e27bf0c’.
 13 #
 14 # Changes to be committed:
 15 # new file: d.txt
 16 #
```

새로운 커밋 메시지를 'PR #2 + PR #4'로 변경하고 저장을 하면 그래프는 다음과 같은 형태로 바뀝니다.

```text
* bdf3e45 - PR #2 + PR #4 - Reid (HEAD -> staging)
* e27bf0c - Merged in feature/add-new-line (pull request #3) - Reid (origin/staging)
| * 83174e6 - Merged in feature/fix-char-order (pull request #5) - Reid (origin/develop, develop)
| * 816108a - Merged in bugfix/add-missing-init-code (pull request #4) - Reid
| * 02f3054 - Merged in feature/add-new-line (pull request #3) - Reid
| * db3cbf3 - Merged in bugfix/fix-typo (pull request #2) - Reid
|/
* fbeca95 - Merged in bugfix/remove-unused-code (pull request #1) - Reid
* cb0ab0e - Add test.txt - Reid (production, origin/production)
```

마지막으로 `staging` 브랜치를 `origin/staging` 브랜치로 push하고, `develop` 브랜치를 `staging`으로 rebase 해서 그래프를 정리해줍니다. `develop` 브랜치에서 `staging`으로 옮긴 커밋은 지금까지 #1, #2, #3, #4 이므로 rebase 작업을 한 후에는 `develop` 브랜치의 #5 커밋만 `staging` 브랜치 끝에 추가 될 것입니다. 정말로 그런지 확인해봅시다.

```sh
$ checkout develop
$ git rebase staging
$ git push origin develop —force
$ git checkout staging
$ git push origin staging
```

```text
* 6709fc5 - Merged in feature/fix-char-order (pull request #5) - Reid (HEAD -> develop, origin/develop)
* bdf3e45 - PR #2 + PR #4 - Reid (staging, origin/staging)
* e27bf0c - Merged in feature/add-new-line (pull request #3) - Reid
* fbeca95 - Merged in bugfix/remove-unused-code (pull request #1) - Reid
* cb0ab0e - Add test.txt - Reid (production, origin/production)
```

깔끔하게 교통 정리가 되었습니다. `develop` 브랜치에 push 한 순서와 관계없이 배포를 원하는 커밋들만 골라서 `staging` 브랜치에 옮겼습니다.

### production 브랜치

`staging` 브랜치가 업데이트되면서 staging 빌드가 만들어졌습니다. 이 빌드로 production 배포 전 통합 테스트를 진행 할 수 있게 되었습니다. 아직 테스트를 완료하지 못했던 PR #5는 staging 빌드에서 제외하였기 때문에 문제가 되지 않습니다. 이제 production 배포를 진행합니다.

```sh
$ git checkout production
$ git merge staging
$ git push origin production
```

```text
* 6709fc5 - Merged in feature/fix-char-order (pull request #5) - Reid (HEAD -> develop, origin/develop)
* bdf3e45 - PR #2 + PR #4 - Reid (staging, origin/staging, production, origin/production)
* e27bf0c - Merged in feature/add-new-line (pull request #3) - Reid
* fbeca95 - Merged in bugfix/remove-unused-code (pull request #1) - Reid
* cb0ab0e - Add test.txt - Reid
```

간단하게 배포가 완료되었습니다.

## Git이 손에 익어야 합니다.

일련의 작업이 복잡해 보일 수 있지만, 사실 git의 핵심적인 역할을 하는 `merge`와 `rebase`에 대한 이해만 충분하면 그리 어려운 일은 아닙니다. 쓸데없이 복잡한 커밋 그래프가 있을 때 interactive rebase를 이용해 원하는대로 붙이고 자르고 더하고 빼는 연습을 하면 익숙해지는데 도움이 됩니다.

그 외에  팁을 몇 개 드립니다.

1. **Git GUI 보다 Git CLI를 이용합니다.** Git CLI로 명령어를 직접 입력하며 작업을 하면 Git GUI 도구로 가려져있던 기능들까지 이해를 할 수 있습니다. Git GUI를 이용하는 것은 터미널에 명령어를 일일이 입력하지 않아도 되므로 편리하지만, 가능하다면 CLI로 동작 방식을 이해하고 사용하는 것이 좋습니다.
2. **`git pull` 명령 대신, `git fetch --prune`과 `git merge` 명령 조합을 사용합니다.** `git pull`은 내부적으로 `git fetch`와 `git merge`를 자동으로 처리해주는 shortcut입니다. 가급적이면 git fetch로 다른 작업자들의 작업 내용을 내려받은 후에 그래프 상황을 보며 merge를 해주는 것이 좋습니다.
3. **의존성 있는 PR은 최대한 지양해야 합니다.** 의존성이 있다면 하나의 PR로 갈 수 있을지를 먼저 의심하고, 어쩔 수 없는 경우라면 반드시 후행 브랜치는 선행 브랜치와 함께 rebase 되어야 합니다.
4. **[bash-git-prompt](https://github.com/magicmonty/bash-git-prompt)와 같은 도구를 이용하여 터미널에서 git 상태를 빠르게 확인 할 수 있게 합니다.** 현재 어느 브랜치에서 작업중인지, staging 되어 있는 파일이 존재하는지, 충돌이 발생했는지 등을 빠르게 파악 할 수 있습니다.
5. **[pretty git log](https://coderwall.com/p/euwpig/a-better-git-log)를 이용하여 터미널에서 git commit graph를 시각적으로 확인 할 수 있게 합니다.** 터미널에서 커밋 그래프를 확인 할 때 반드시 필요한 도구입니다.
6. **공동 작업 중인 topic branch에 대한 rebase는 상대 작업자를 곤궁에 빠뜨릴 수 있습니다.** rebase는 원래 있던 커밋을 없애고 새로운 커밋을 만드는 파괴적인 작업입니다. 여러명이 작업중인 브랜치를 강제로 rebase 해버린다면 어떤 작업자는 이미 사라진 커밋 위에서 작업을 해야 할 수 있습니다.

## 마치며

여기서 소개하는 `staging` 브랜치는 `develop` 브랜치와 `production` 브랜치 사이에 존재하는 쿠션입니다. `develop` 브랜치를 production ready 상태로 만들기 위해 개발자들이 신경을 곤두세우지 않아도 됩니다. 배포에 신경쓰기 보다 개발에 집중 할 수 있는 환경을 만듭니다. `develop` 브랜치로부터 선택되어 `staging`으로 옮겨진 커밋들이야 말로 production ready 상태를 확실히 검증해야 합니다. `staging` 브랜치로부터 빌드 된 staging 테스트 환경은 develop 환경과 달리 production 환경과 동일하게 구성하여 side-effect가 없는 깨끗한 상태에서 테스트가 이루어지도록 해야합니다.

이 workflow에서 핵심은 `staging` 브랜치의 운용입니다. 단지 git 브랜치만 따로 관리하는 것이라면 큰 의미가 없습니다. 프로덕트를 구성하는 컴포넌트들이 스스로 언제든지 배포 가능한 빌드를 만들어 낼 수 있고, 이들을 통합하여 stagnig 테스트 환경을 구축 할 수 있어야 합니다. 각 컴포넌트의 개발자가 할 수 있는 최대한의 테스트와 리뷰를 거쳐 올라온 기능이 개발자의 손을 떠나 통합 시스템의 일부로서 올바르게 동작하는지를 최종 검증 할 수 있어야 합니다.

지속적 통합과 배포의 시기가 도래하면서 '배포'를 위해 예전만큼 큰 노력을 들일 필요가 없게 되었습니다. 대신 언제든 퀄리티 있는 빌드를 실행 할 수 있게 해야 하는 책임은 더 커졌습니다. 여러분의 프로젝트에 지속적으로 안정감 있는 빌드를 배포하는데 이 글이  도움이 되었으면 좋겠습니다.


