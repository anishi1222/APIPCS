# API Platform Cloud Service Tutorial

API Platform Cloud Service（APIPCS）のチュートリアルです。

このチュートリアルではAPIPCSの環境設定、API開発などに必要なタスクを辿っていきます。詳細は[APIの設計](./tutorials/design/design_api)をご覧ください。

> 注：リンクは常に新しいタブで開いてください（右クリック→*新しいタブでリンクを開く*）。そうすれば、タスクを完了後にリンク先のチュートリアルなどからこの演習ガイドに戻ってくることができます。

## 進め方

このトレーニングの全体構成は以下のようになっています。

- シナリオ
  - チュートリアル
    - スクリーンキャスト（近日公開）

シナリオを開始する際に、リンク先のチュートリアルを開いてガイダンスを参照することもできますが、盲目的にチュートリアルの手順に従うのではなく、必要なガイダンスだけで操作するようにしてください。実際には、変更を加える際にAPIをデプロイするなど、自然に繰り返されるいくつかのタスクがあります。最初のデプロイ時にはチュートリアルを使う必要があるかもしれませんｇ、2回目以後は独力で試してください。

## Getting Started

1. 必要な環境を選択します。
    1. [環境](./environments) に従って選択する
    1. [シナリオ](./scenarios) に従って利用したいシナリオを選択する（こちらは現時点で1個のみなので選択は簡単ですね）
  
> 注：リンクは常に新しいタブで開いてください（右クリック→*新しいタブでリンクを開く*）。そうすれば、タスクを完了後にリンク先のチュートリアルなどからこの演習ガイドに戻ってくることができます。

## Trainers

When conducting and instructor-led event, there are a few steps that you need to complete in order to prepare.

### Pre-event setup

1. Choose the environment you will use
    1. You can visit [Environments](./environments) to learn about the options
    1. Create a ppt, or other medium that provides an overview of API Platform and this training.
        1. In your ppt, you can include the getting *Accessing Content* section below, or plan to either write it up a whiteboard, etc, or pull up this page to present to your audience.
  
### Kick-off the event

1. Introduce API Platform and the training
    1. If you chose the *Demo/Training Instance*, you should assign sequence numbers to your attendees.  Bear in mind that they may need to add additional identifiers to properly separate their work.
1. Direct your attendees to scenario in Developer Cloud Service by completing the following:
    1. Point your browser to: http://bit.ly/APIWarrantyClaim
        1. When asked for the domain, enter: *partnerpaas18*
        1. Log in as user: *apipartner*
        1. The password is: *Aut0nomou5!* (Aut-zero-nomou-five-!)

#### Using the Demo/Training Instance

It is best to provision your own instance using GSE, trial or other means, but we do maintian a *PM demo instance*

The *PM demo instance* is the simplest to use, but you should be aware that anybody can use it!  You need to come up with a way to segment your attendees work.

If you plan to use the *PM demo instance*, you will need to assign a sequence number to each attendee.  Each attendee will append this sequence to the username and to the password.  For example an attendee who has been assigned the sequence number of "1",  would log in as Follows:

- User: api-manager-user-1
- Password: OracleiPa$$Us3r1

> Note: You should come up with an additional string to further segment the work in your event, such as the city/location, etc.  For Example, if I were presenting this in Orlando and their student sequence, I might advise my students to add "Orlando" (e.g. TicketService**Orlando1**)

## Questions

If you have any questions about these tutorials or API Platform CS, follow the path below:

### Oracle Employees

1. Visit `http://my.oracle.com/go/apipcs` (VPN required) for a wealth of information about API Platform CS
1. Use the FAQ that you will find on the above site.
1. If your questions is not answered, then use the distribution list, also at the above site.

### Non-Oracle Employees

1. In the Developer Cloud Service instance where you are accessing this content, navigate to Issues.  Create a new Issue, describing your issue/question.  Make sure you provide your e-mail address in the description and assign the issue to *Robert Wunderlich*