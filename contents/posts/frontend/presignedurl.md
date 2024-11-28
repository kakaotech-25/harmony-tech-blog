---
title: AWS S3 Presigned URL을 활용한 React 파일 업로드 구현
date: "2024-11-28"
writer: 리안
tags:
  - React
  - Next
  - Typescript
  - AWS S#
  - Presigned URL
  - 리안
previewImage: s3.png
---

면접왕 김개발 프로젝트를 진행하며 사용자가 업로드한 자기소개서, 이력서 등을 분석해야 하는 과정이 있어서 파일을 입력받아 전달해주는 단계가 필요했다.
여러 논의 끝에 AWS S3에 파일을 올리고 Presigned URL을 받아서 처리하는 방식으로 하기로 결정했다.

## 왜 Presigned URL을 사용할까?

파일을 일일히 백으로 보내는 방식보다는 클라이언트에서 직접 S3에 파일을 업로드하면 서버 부담이 줄어들고 업로드 속도가 빨라진다.  
하지만 이 방법을 presigned url 없이 하면 S3 버킷을 퍼블릭엑세스로 전환해야 하는데 이렇게 하면 보안에 취약하다는 단점이 있다.  
Presigned URL은 이를 해결하기 위해 사용된다.  
Presigned URL은 S3 객체에 대한 제한된 시간 동안의 액세스를 허용하는 URL로 퍼블릭엑세스를 허용할 필요가 없어 보안이 강화되고 클라이언트에서 직접처리하여 서버부담이 감소되고 구현이 간단하다는 장점이 있다.

## React에서 Presigned URL을 사용한 파일 업로드 흐름

1. 사용자가 업로드할 파일을 선택한다.
2. 프론트엔드에서 파일 이름과 타입을 백엔드로 전달하고 생성된 Presigned URL을 전달받는다.
3. 받은 URL을 사용해 파일을 S3에 직접 업로드한다.
4. 업로드 사항을 사용자에게 알려준다. (면접왕 김개발의 경우에는 토스트로 업로드여부를 안내하고 미리보기 기능을 제공한다.)

## 구현과정

### Presigned URL 요청

```ts
const handleUpload = async (uploadFile: File) => {
  if (!uploadFile) {
    toast.error("파일이 선택되지 않았습니다.")
    return
  }

  setUploading(true)

  try {
    const response = await fetch("/api/s3/uploadFiles", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        fileName: `cover-letters/${uploadFile.name}`,
        fileType: uploadFile.type,
      }),
    })

    if (!response.ok) {
      throw new Error("Presigned URL 요청 실패")
    }

    const { presignedUrl } = await response.json()
    await uploadToS3(presignedUrl, uploadFile)
  } catch (error) {
    toast.error("Presigned URL 요청 또는 업로드 실패")
  } finally {
    setUploading(false)
  }
}
```

fileName과 fileType을 전달하고 백엔드로부터 JSON 형식의 presignedURL을 받아온다.

### S3에 파일 업로드

```ts
const uploadToS3 = async (presignedUrl: string, file: File) => {
  try {
    await fetch(presignedUrl, {
      method: "PUT",
      headers: { "Content-Type": file.type },
      body: file,
    })

    toast.success("파일 업로드 성공!")
  } catch {
    toast.error("S3 업로드 중 오류가 발생했습니다.")
  }
}
```

fetch 를 사용해서 파일을 S3에 업로드하고 업로드 성공여부를 토스트를 통해 사용자에게 알려준다.

## 구현하여 느낀점

그 전에는 파일을 무작정 백으로 전달하는 방식으로만 구현을 했었는데 보안도 신경쓰고 서버 부담도 최소화할 수 있는 방법을 알게되었고 적용해보는 경험이 되었다.
옛날에 네이버블로그에 글을 올리고 그 사진을 복사해와서 새로운 글을 작성한 후 며칠이 지나 들어가보니 존재하지 않는 사진이라고 떠있던 적이 있다. 그저 오류라고 생각했던 부분인데 그때 사진이 만료되었던게 바로 이 presignedURL 때문이라는 것을 알게 되었다.

S3에 저장할 때는 같은 이름의 파일이 들어오면 (1) 이런식으로 따로 저장되는게 아니라 덮어쓰기로 저장이 되기 때문에 이 부분을 꼭 고려해야 한다.
현재 자기소개서, 이력서를 올리는 파일업로드 뿐만아니라 프로필이미지 또한 S3에 저장하고 있는데 이때는 이런 덮어쓰기 방식을 사용하고 있다.
과거의 사진은 보관하지 않고 바뀌는 사진으로 변경하는 방식으로 구현할 예정이라 이때에는 파일명을 profileimg.png 로 통일해서 주면 될듯하다.
