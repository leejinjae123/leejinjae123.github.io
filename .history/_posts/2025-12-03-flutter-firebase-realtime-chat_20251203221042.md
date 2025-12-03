---
title: "Flutter Firebase 실시간 채팅 구현"
date: 2025-12-03
last_modified_at: 2025-12-03
categories:
  - Flutter
tags:
  - Flutter
  - Firebase
  - Firestore
  - 실시간 채팅
sidebar:
  nav: "sidebar-category"
---

## Firebase Cloud Database란?

- Cloud 방식의 실시간 Database로, Flutter에서 StreamBuilder로 실시간 변경사항을 적용 할 수 있다.
- MySql이나 MSSQL 등은 RDBMS라는 관계형 데이터베이스 구조를 사용하는 반면,
    
    Firestore이나 MongoDB의 경우 문서형 데이터 베이스를 기본 구조로 하기때문에, 
    
    컬렉션 - 문서 - 필드의 구조를 가지고 있다.
    
- Cloud Firestore의 구조

    ![Firebase Cloud Firestore 구조](/assets/png/Firebase_Cloud_Firestore_%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9.png)
    

## 그래서 어떻게 쓰겠다는 건데?

💡 Flutter 프로젝트 내에서 Firestore에 직접 관여할 수 있는 쿼리문을 적용하고,
	 StreamBuilder을 이용해 Firestore의 DB를 실시간으로 화면에 구현한다면
	 채팅기능을 구현할 수 있다.

- Firebase의 프로젝트에서 Cloud Firestore을 시작.
- 내가 구현할 채팅방의 경우, 첫페이지는 User가 받은 채팅의 목록들을 뿌려주고,
    
    채팅의 목록중 하나를 클릭할 경우 상대방과 나눈 채팅목록을 모두 보여주는 형식으로 구현하였다.
    
- Database 구조는 컬렉션을 두개로 나누어
    - User의 UID를 컬렉션의 ID로 받아, User가 받은 채팅의 정보를 모으는 컬렉션
    
    ![스크린샷 2023-04-25 오전 11.38.04](/assets/png/%25E1%2584~1.PNG)
    
    - 모든 유저가 쓴 채팅창을 보관하는 Chat 컬렉션을 구조로 잡았다.
    
    ![스크린샷 2023-04-25 오전 11.38.41](/assets/png/%E1%84%80~2.PNG)
    
- 채팅방 첫 페이지(User 채팅방)

```dart
// 로그인 한 유저 정보
final user = FirebaseAuth.instance.currentUser;
// FireStore 접근
final db = FirebaseFirestore.instance;

SliverToBoxAdapter(
  child: user != null
    ? Container(
      padding: const EdgeInsets.all(Dimensions.HOME_PAGE_PADDING),
      decoration: BoxDecoration(
        border: Border.symmetric(
          horizontal: BorderSide(color: ColorResources.HINT_TEXT_COLOR)
        ),
      ),
      child: StreamBuilder<QuerySnapshot>(
				// user.uid와 일치하는 컬렉션의 필드 호출
        stream: db.collection(user!.uid).snapshots(),
        builder: (context, AsyncSnapshot<QuerySnapshot> snapshot) {
          if (snapshot.hasError) {
            return Text('Error : ${snapshot.error}');
          } else if (snapshot.hasData == false) {
						// 필드가 없을 시
            return Text('채팅이 없습니다.');
          } else if (snapshot.connectionState == ConnectionState.waiting) {
						// 채팅창 데이터를 받아오는 과정일 시 로딩 애니메이션 출력
            return Center(child: CircularProgressIndicator());
          }
					// 받아온 필드를 List구조화
          final chatList = snapshot.data!.docs;
          return Column(
            children: List.generate(chatList.length, (index) {
              return Row(
                children: [
                  Flexible(
                    flex: 1,
                    child: Center(
                      child: Text('이미지'),
                    ),
                  ),
                  Flexible(
                    flex: 3,
                    child: InkWell(
                      onTap: () {
                        Navigator.push(
                          context,
                          MaterialPageRoute(
                            builder: (context) =>
															// 'LinkUID' 필드의 값을 data로 ChatDetail 페이지에 보냄
                              ChatDetail(data: chatList[index]['LinkUID'])
                          )
                        );
                      },
                      child: Column(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
													// 필드 'name'의 값
                          Text(chatList[index]['name']),
                          SizedBox(height: 10),
													// 필드 'content'의 값
                          Text(chatList[index]['content']),
                        ],
                      ),
                    ),
                  ),
                ],
              );
            }),
          );
        }
      ),
    )
    : Container(
      child: Text('로그인을 해주세요'),
    ),
),
```

- 채팅 상세보기 페이지(1대1 대화방)

```dart
// TextField 컨트롤러
final TextEditingController txc = TextEditingController();
// 로그인 한 user정보 받아오기
final user = FirebaseAuth.instance.currentUser;
String text = '';
// Firestore 연결
final db = FirebaseFirestore.instance;

// 채팅을 입력 후 보내기 버튼을 누르면 실행되는 함수
void sendMessage() async {
			// 포커스 아웃
      FocusScope.of(context).unfocus();
      if (text.trim().isEmpty){
        print('error');
      }else{
				// 메세지를 보낼시, user의 채팅방의 최신화
        db.collection(data).doc(user!.uid).set({
          'LinkUID' : user!.uid,
          'name' : 'user!.name',
          'content' : text,
          'Time' : Timestamp.now(),
        });
				// 메세지를 보낼시, 상대방의 채팅방의 최신화
        db.collection(user!.uid).doc(data).set({
          'LinkUID' : data,
          'name' : 'user!.name',
          'content' : text,
          'Time' : Timestamp.now(),
        });
				// 전체 채팅 목록 추가
        db.collection('chat').add({
          'ID' : [user!.uid, data],
          'content' : text,
          'Time' : Timestamp.now(),
        });
      }
			// 텍스트필드 초기화
      txc.clear();
    }

SliverToBoxAdapter(
  child: user != null
      ? Container(
          margin: const EdgeInsets.all(Dimensions.HOME_PAGE_PADDING),
          child: StreamBuilder(
							// 필드 ID 의 배열이 정확하게 [user!.uid, data]이거나 [data,user!.uid]인
								 ID가 포함된 값을 가져온다.
							// 이렇게 표현하는 이유는 쿼리에 OR절이 없기때문이다.
              stream: db.collection('chat').where('ID', whereIn: [
                [user!.uid,data],[data,user!.uid]
              ]).orderBy('Time',descending: true).snapshots(),
              builder: (context, AsyncSnapshot<QuerySnapshot> snapshot) {
                if (snapshot.hasData == false) {
                  return Text('데이터가 없음');
                } else if (snapshot.hasError) {
                  return Text('Error : ${snapshot.error}');
                }
                if (snapshot.connectionState == ConnectionState.waiting) {
                  return Center(child: CircularProgressIndicator());
                }
                final chatList = snapshot.data!.docs;

                return Column(
                    children: List.generate(chatList.length, (index) {
									// TimeStamp로 표기되어있는 'Time'필드를 DataTime으로 변경
                  DateTime time = chatList[index]["Time"].toDate();

                  return Row(
										// Chat Bubble의 위치를 본인과 상대방의 시작점을 다르게 한다.
                    mainAxisAlignment: chatList[index]["ID"][0] == user!.uid
                        ? MainAxisAlignment.end
                        : MainAxisAlignment.start,
                    children: [
                      Column(
                        crossAxisAlignment: CrossAxisAlignment.end,
                        children: [
                          Container(
                            margin: const EdgeInsets.symmetric(
                                vertical: Dimensions.PADDING_SIZE_LARGE),
                            padding: const EdgeInsets.fromLTRB(
                              Dimensions.PADDING_SIZE_SMALL,
                              Dimensions.PADDING_SIZE_EXTRA_SMALL,
                              Dimensions.PADDING_SIZE_SMALL,
                              Dimensions.PADDING_SIZE_EXTRA_SMALL,
                            ),
                            constraints: BoxConstraints(
                                maxWidth:
                                    MediaQuery.of(context).size.width / 1.7),
                            decoration: BoxDecoration(
                                borderRadius: BorderRadius.circular(15),
                                border: Border.all(
                                    color: ColorResources.CHAT_ICON_COLOR)),
                            child: Text(
                              chatList[index]['content'],
                              style: TextStyle(
                                  fontSize: Dimensions.FONT_SIZE_DEFAULT),
                              maxLines: 1000,
                              overflow: TextOverflow.ellipsis,
                            ),
                          ),
                          Text(DateFormat('yyyy-MM-dd HH:mm:ss').format(time)),
                        ],
                      ),
                    ],
                  );
                }).reversed.toList());
              }),
        )
      : Container(
          child: Text('로그인을 해주세요'),
        ),
),
// 채팅 입력 TextField
bottomSheet: Container(
  margin: EdgeInsets.all(Dimensions.HOME_PAGE_PADDING),
  child: Row(
    children: [
      Expanded(
        child: TextField(
          controller: txc,
          onChanged: (val) {
            setState(() {
              text = val;
            });
          },
          decoration: InputDecoration(
            hintText: '채팅을 입력해주세요',
          ),
        ),
      ),
      IconButton(onPressed: sendMessage, icon: Icon(Icons.send))
    ],
  ),
),
```

