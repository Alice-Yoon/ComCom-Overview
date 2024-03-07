# ComCom-Overview

뉴스, 코인, 주식, 유머 등의 주제로 유저들이 소통하는 커뮤니티 플랫폼.

풀스택 개발자로서 프론트엔드와 백엔드 API를 전담하여 개발하였습니다.

-----

## Overview.

### Mobile:

[<img src="https://img.youtube.com/vi/O_nE4jP34H0/hqdefault.jpg" width="600" height="300"
/>](https://www.youtube.com/embed/O_nE4jP34H0)

### PC:

[<img src="https://img.youtube.com/vi/hpHpoA0m6mQ/hqdefault.jpg" width="600" height="300"
/>](https://www.youtube.com/embed/hpHpoA0m6mQ)

<br/>

## 주요 기능.

- **회원가입/비밀번호 찾기 - 이메일 인증방식 (node-mailer)**
    
[<img src="https://img.youtube.com/vi/h3jAhAf5PtU/hqdefault.jpg" width="600" height="300"
/>](https://www.youtube.com/embed/h3jAhAf5PtU)
    
- **제목, 작성자 닉네임, 댓글내용으로 게시글 검색**
    
[<img src="https://img.youtube.com/vi/KzthyTvqZfE/hqdefault.jpg" width="600" height="300"
/>](https://www.youtube.com/embed/KzthyTvqZfE)
    
- **게시글 작성, 스크랩, 좋아요/싫어요 처리**
    
[<img src="https://img.youtube.com/vi/GLce1Pw1bO0/hqdefault.jpg" width="600" height="300"
/>](https://www.youtube.com/embed/GLce1Pw1bO0)
    
- **댓글작성**
    
[<img src="https://img.youtube.com/vi/MVlECTF4VHM/hqdefault.jpg" width="600" height="300"
/>](https://www.youtube.com/embed/MVlECTF4VHM)
    
- **게시글에 댓글 달린 경우 알림 받기**
    
[<img src="https://img.youtube.com/vi/IwiRxRBi_c4/hqdefault.jpg" width="600" height="300"
/>](https://www.youtube.com/embed/IwiRxRBi_c4)

- **포인트 시스템: 활동 포인트 적립, 유저끼리 포인트 선물**
    
[<img src="https://img.youtube.com/vi/NtY__dOZ9A4/hqdefault.jpg" width="600" height="300"
/>](https://www.youtube.com/embed/NtY__dOZ9A4)

<br/>

-----

## [FE] React-Quill 에디터 이미지 처리

이미지 업로드 시 자동으로 base64로 인코딩되어 url이 길어지는 문제를 커스텀 Image Handler 함수를 만들어 Firebase에 이미지를 저장하고 저장된 url을 사용하여 해결하였습니다.

```jsx
const imageHandler = () => {  
    const input = document.createElement('input');
    input.setAttribute('type', 'file');
    input.setAttribute('accept', 'image/*');
    input.click();
  
    input.addEventListener('change', async (e) => {
      try {
        // Save the image in firebase
        const image = e.target.files[0]
        const { url } = await imageUpload(image)
    
        // Insert image into the editor
        const editor = quillRef.current.getEditor()      
        const cursor = editor.getSelection(true) 
        editor.insertEmbed(cursor.index, 'image', url)

				// Move the cursor to the right side of the image for better user experience
        editor.setSelection(cursor.index + 1) 

      } catch (error) {
        const editor = quillRef.current.getEditor()
        const cursor = editor.getSelection(true)
        editor.deleteText(cursor.index, 1)
      }
      
    })
  }
```

## [BE] auth 미들웨어

'protect'이라는 미들웨어를 생성해서 로그인 필요한 route 접근시 로그인 여부를 체크하게 하였습니다.

```jsx
exports.protect = asyncHandler(async (req, res, next) => {
  const authHeader = req.headers.authorization

  if(!authHeader) {
    return next(new ErrorResponse('You do not have access permission.', 401))
  }

  const accessToken = authHeader.split(' ')[1]
  const refreshToken = req.cookies.refreshToken

  if(!accessToken || !refreshToken) {
    return next(new ErrorResponse('You do not have access permission.', 401))
  }

  try {
    const decoded = await jwt.verify(accessToken, process.env.JWT_SECRET)
    const user = await DB.User.findOne({ where: { id: decoded.id } })
    req.user = user
    next()
  } catch (error) {
    return next(new ErrorResponse('You do not have access permission.', 401))
  }
});
```

```jsx
router
    .route('/')
		.post(protect, addPost)
```

## 댓글에서 다른 유저 태그하고, 태그시 알림 받기

유저들은 다른 유저를 댓글에서 태그할 수 있고, 태그된 유저는 알림을 받습니다.

```jsx
// Front-End:

const getTaggedNickname = () => {
    const regex = /@[^\s]+/g
    const matchedWords = comment.match(regex)

    const formatted = matchedWords ? matchedWords.map(word => word.replace('@', '')) : []
    return formatted
}

const submitHandler = () => {
    const taggedNicknames = getTaggedNickname()
		...
}
```

```jsx
// Back-End:

const tags = comment.tags

if(tags.length) {
    const users = await DB.User.findAll({ 
        where: { 
           nickName: { [Sequelize.Op.in]: tags }
		    }
		}
)
            
const userAlarms = users.map(user => ({
    userId: user.dataValues.id,
    type: 'comment',
    targetId: req.body.postingId,
    targetPreview: formatStringLength(post.dataValues.title, ALARM_PREVIEW_TEXT_LENGTH),
     moveTo: `/post/${req.body.postingId}`
}))
        
const alarms = [...alarms, ...userAlarms] 

        
await DB.UserAlarm.bulkCreate(alarms, { transaction })
```
