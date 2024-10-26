---
title: 랭체인(2)
date: "2024-10-25"
writer: mindy
series: langchain study
tags:
  - ai
  - LangChain
  - mindy
previewImage: langchain.png
---

> 💡 현재 포스트는 harmony 팀 크루 [mindy](https://github.com/1013115) 가 작성했습니다.

저번 랭체인 포스트에 이어 작성합니다. 랭체인에 있는 6개의 모듈 중 model I/O에 대해 알아보려고 합니다.


# Model I/O
## 1. Language Models
### 1-1. LLMs
하나의 텍스트 입력에 대해 하나의 텍스트 출력을 반환하는 전형적인 대규모 언어 모델을 다루는 모듈이다. 주로 단일 요청에 대한 복잡한 출력을 생성하는데 적합하다. 광범위한 언어 이해 및 텍스트 생성 작업에 사용된다. 예를 들어, 문서요약, 콘텐츠 생성, 질문에 대한 답변 생성 등 복잡한 자연어처리 작업을 수행할 수 있다. 랭체인에서 직접 제공하는 것이 아니라, openai, Cohere, Huggingface 등과 같은 다양한 LLM 제공 업체로부터 모델을 사용할 수 있게하는 플랫폼 역할을 한다. 
```python
from langchain_openai import OpenAI
llm = OpenAI()
llm.invoke("한국의 대표적인 관광지 3곳을 추천해주세요.")
```
### 1-2. ChatModels
단순히 하나의 텍스트를 입력하는 것이 아니라, 채팅 형식의 대화를 입력하면 응답을 받을 수 있도록 되어있다. OpenAI의 Chat Completions API를 지원하기 위해 만들어진 랭체인의 모듈이 `ChatModel`이다. 사용자와의 상호작용을 통한 연속적인 대화 관리에 더 적합하다. 랭체인은 OpenAI, Cohere, Hugging Face 등 다양한 모델 제공 업체와의 통합을 지원한다. 이를 통해 개발자는 여러 소스의 ChatModels를 조합하여 활용할 수 있다. 또한, 다양한 작동 모드를 지원한다. 동기(sync), 비동기(async), 배치(batch), 스트리밍(streaming) 모드에서 모델을 사용할 수 있는 기능을 제공한다. 다양한 애플리케이션 요구사항과 트래픽 패턴에 따라 유연한 대응이 가능하다. 
```python
from langchain_openai import ChatOpenAI
from langchain_core.messages import AIMessage, HumanMessage, SystemMessage

chat = ChatOpenAI()

messages = [
	SystemMessage(content = "You are a helpful assistant."),
	HumanMessage(content = "안녕하세요! 저는 민디라고 합니다!"),
	AIMessage(content = "안녕하세요 민디씨! 어떻게 도와드릴까요?"),
	HumanMessage(content="제 이름을 아시나요?"),
]

result = chat.invoke(messages)
print(result.content)
```
랭체인의 `SystemMessage`, `HumanMessage`, `AIMessage`는 각각 Chat Completions API의 “role":"system", "role":"user", "role":"assistant" 에 대응한다. 
#### langchain LLM 모델 파라미터 설정
랭체인에서는 모델 생성 단계, 모델 호출 단계, bind 메소드를 사용하여 파라미터를 지정할 수 있다. bind 메소드는 특정 모델 설정을 기본값으로 사용하고자 할 때 유용하며, 특수한 상황에서 일부 파라미터를 다르게 적용하고 싶을 때, 사용한다.
```python
from langchain_openai import ChatOpenAI
 
# 1. 모델 생성 단계
params  = {
	"temperature" : 0.7,  # 생성된 텍스트의 다양성 조정
	"max_tokens" : 100,   # 생성할 최대 토큰 수
}

kwargs = {
	"frequency_penalty" : 0.5,  # 이미 등장한 단어의 재등장 확률
	"presence_penalty" : 0.5,   # 새로운 단어의 도입을 장려
	"stop" : ["\n"]             # 정지 시퀀스
}

model = ChatOpenAI(model="gpt-4o-mini", **parmas, model_kwargs = kwargs)

question = "태양계에서 가장 큰 행성은 무엇인가요?"
response = model.invoke(input=question)

# 2. 모델 호출 단계
params = {
	"temperature": 0.7,
	"max_tokens" : 10,
}

response = model.invoke(input = question, **params)

# 3. bind 메소드
model = ChatOpenAI(model = "gpt-4o-mini", max_tokens =100)
chain = prompt | model.bind(max_tokens=10)
response = chain.invoke()
```
## 2. Prompt
다음으로는 LLM을 이용한 애플리케이션 개발에서 매우 중요한 요소인 prompt이다. prompt란 사용자가 제공하는 지침이나 입력의 집합으로, 모델의 응답을 안내하고 문맥을 이해하며 질문에 답하거나 문장을 완성하거나, 대화를 나누는 등 관련성 있고 일관된 언어 기반 출력을 생성하는데 도움을 주는 역할을 한다. 
langchain에서 제공하는 prompt template은 다음과 같다. 그 중 몇가지만 살펴보도록한다. 
```python
from langchain.prompts import (
    PromptTemplate,
    PipelinePromptTemplate,
    MessagesPlaceholder,
    ChatPromptTemplate,
    HumanMessagePromptTemplate,
    SystemMessagePromptTemplate,
    AIMessagePromptTemplate,
    FewShotChatMessagePromptTemplate,
    FewShotPromptWithTemplates
)
```
### 2-1. PromptTemplate
이름 그대로 PromptTemplate를 사용하면 프롬프트를 템플릿화할 수 있다. LLMs에 메시지를 전달하기 전에 문장 구성을 편리하게 만들어준다. 
```python
from langchain_core.prompts import PromptTemplate

template = """
안녕하세요, 제 이름은 {name}이고, 나이는 {age}입니다.
"""
prompt_template = PromptTemplate(
	input_variables = ["name","age"],
	template = template,
)
result = prompt.format(name="홍길동", age=30)
result
```
promptTemplate은 프로그램에서 문자열의 일부를 대체하는 것일뿐, 내부적으로 LLM을 호출하는 일은 하지 않는다.

### 2-2. ChatPromptTemplate
promptTemplate를 Chat Completions API의 형식에 맞게 만든 것이 ChatPromptTemplate이다. 
입력은 여러 메시지를 원소로 갖는 list 로 구성되며, 각 메시지는 역할(role)과 내용(content)으로 구성된다. ChatPromptTemplate을 작성하는 방법에는 `Message`유형과 `2-tuple` 형태의 메시지 리스트 크게 2가지로 나눌 수 있다. 
#### (1) Message 유형
- SystemMessage: 시스템의 기능 설명, Chat Completions API의 `“role":"system"`에 대응
- HumanMessage: 사용자의 질문을 나타냄, Chat Completions API의 `“role":"user"`에 대응
- AIMessage: AI 모델의 응답 제공,  Chat Completions API의 `“role":"assistant"`에 대응
- FunctionMessage: 특정 함수 호출의 결과를 나타냄
- ToolMessage: 도구 호출의 결과를 나타냄
```python
from langchain_core.prompts import HumanMessagePromptTemplate, SystemMessagePromptTemplate

chat_prompt = ChatPromptTemplate.from_messages([
    SystemMessagePromptTemplate.from_template("이 시스템은 {project} 질문에 답변할 수 있습니다."),
    HumanMessagePromptTemplate.from_template("{user_input}")
])

messages = chat_prompt.format_messages(project="천문학", user_input="태양계에서 가장 큰 행성은 무엇인가?")
messages = chat_prompt.format_prompt(project="천문학", user_input="태양계에서 가장 큰 행성은 무엇인가?").to_messages()
messages
```
> ✓ from_messages 함수는 langchain에서 대화형 에이전트와 상호작용하는데 필요한 메시지 객체를 생성하는데 사용된다. 주로 사용자가 이전에 주고받은 메시지 이력이나 새로운 입력 메시지를 통일된 형식으로 변환할 때 자주 사용한다. 

> ✓ format_messages 함수는 다양한 입력 값을 메시지 형식으로 포맷하는데 사용된다. 주로 파라미터가 여러 개 있을 때, 그 파라미터들을 템플릿에 맞춰 하나의 메시지로 포맷하거나 형식화한다.
#### (2) 2-tuple 형태의 메시지 리스트
ChatPromptTemplate.from_messsage 메서드를 사용하여 메시지 리스트로부터 ChatPromptTemplate 인스턴스를 생성하는 이 방식은 대화형 프롬프트를 생성하는데 유용하다. 각 메시지의 역할(type)과 내용(content)을 기반으로 프롬프트를 구성한다. 
```python
from langchain_core.prompts import ChatPromptTemplate

chat_prompt = ChatPromptTemplate.from_messages([
    ("system","이 시스템은 천문학 질문에 답변할 수 있습니다."),
    ("user","{user_input}"),
])

messages = chat_prompt.format_messages(user_input="태양계에서 가장 큰 행성은 무엇인가?")
messages
```
💡 Chat Completions API는 openAI에서 사용하는 api로 형식은 아래와 같다
~~~python
from openai import OpenAI
client = OpenAI()
completion = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {
            "role": "user",
            "content": "Write a haiku about recursion in programming."
        }
    ]
)

print(completion.choices[0].message)
~~~~
### 2-3. Few-Shot Prompt
언어모델에 몇가지 예시를 제공하여 특정 작업을 수행하도록 유도하는 기법이다. 이를 통해, 모델의 성능을 크게 향상시킬 수 있다. 또한, 주어진 예제들을 참고하여 더 정확하고 일관된 응답을 생성할 수 있다.
~~~python
from langchain_core.prompt import FewShotPromptTemplate

example_prompt = PromptTemplate.from_template("질문 : {question}\n{answer}")
examples = [
    {
        "question": "지구의 대기 중 가장 많은 비율을 차지하는 기체는 무엇인가요?",
        "answer": "지구 대기의 약 78%를 차지하는 질소입니다."
    },
    {
        "question": "광합성에 필요한 주요 요소들은 무엇인가요?",
        "answer": "광합성에 필요한 주요 요소는 빛, 이산화탄소, 물입니다."
    },
    {
        "question": "피타고라스 정리를 설명해주세요.",
        "answer": "피타고라스 정리는 직각삼각형에서 빗변의 제곱이 다른 두 변의 제곱의 합과 같다는 것입니다."
    }
    ...
]
from langchain_core.prompts import FewShotPromptTemplate

prompt = FewShotPromptTemplate(
	examples = examples,
	example_prompt = example_prompt,  # 예제 포맷팅에 사용할 접미사
	suffix = "질문 : {input}",   # 예제 뒤에 추가될 접미사
	input_variable=["input"],   # 입력 변수 저장
)

print(prompt.invoke({"input":"화성의 표면이 붉은 이유는 무엇인가요?"})).to_string()
~~~
> from_template 함수는 langchain에서 템플릿을 기반으로 메시지를 생성하는 함수이다. 주어진 템플릿에 따라 변수나 입력값을 채워 넣어 메시지를 생성하는데 사용된다. 기본적으로 메시지 템플릿을 사전에 정의하고, 필요한 변수만 나중에 전달해 자동으로 메시지를 만들 수 있다.  

`example_selectors`를 사용하여 예제가 많은 경우 프롬프트에 포함할 예제를 선택해야할 때 사용하는 클래스이다. 사용자가 입력한 내용에 가까운 예시를 자동으로 선택하여 삽입할 수 있다. 
~~~python
from langchain_chroma import Chroma
from langchain_core.examples_selectors import SemanticSimilarityExampleSelector
from langchain_openai import OpenAIEmbeddings

example_selector = SemanticSimilarityExampleSelector.from_examples(
	examples,
	OpenAIEmbeddings(),
	Chroma,  # 벡터 저장소, 추후에 다시 나올 예정
	k=1,  # 선택할 예제 수
)

question = "화성의 표면이 붉은 이유는 무엇인가요?"
selected_examples = examples_selector.select_examples({"question":question})
print(f"입력과 가장 유사한 예제 : {question}")
for example in selected_examples:
	print("\n")
	for k, v in example.items():
		print(f"{k}:{v}")
~~~
### 2-4. Partial Prompt
필요한 값의 일부만 미리 입력하여 새로운 프롬프트 템플릿을 만드는 방법이다. 복잡한 프롬프트를 관리하거나, 동적인 값을 효율적으로 처리해야할 떄 유용하다. 여기에는 함수를 이용한 부분 포맷팅과 문자열 값을 이용한 부분 포맷팅 2가지가 존재한다.
~~~python
# 1. 문자열 값을 사용한 부분 포맷팅
from langchain_core.prompts import PromptTemplate

prompt = PromptTemplate.from_template("지구의 {layer}에서 가장 흔한 원소는 {element}입니다.")

# 'layer' 변수에 '지각' 값을 미리 지정하여 부분 포맷팅
partial_prompt = prompt.partial(layer="지각")

# 나머지 'element' 변수만 입력하여 완전한 문장 생성
print(partial_prompt.format(element = "산소"))

prompt = PromptTemplate(
	template = "지구의 {layer}에서 가장 흔한 원소를 {element}입니다.",
	input_variables = ["element"],  # 사용자 입력이 필요한 변수
	partial_variables = {"layer":"맨틀"}  # 미리 지정된 부분 변수
)
print(prompt.format(element="규소"))

# 2. 함수를 사용한 부분 포맷팅
from datetime import datetime

# 현재 계절을 반환하는 함수 정의
def get_current_season():
    month = datetime.now().month
    if 3 <= month <= 5:
        return "봄"
    elif 6 <= month <= 8:
        return "여름"
    elif 9 <= month <= 11:
        return "가을"
    else:
        return "겨울"

# 함수를 사용한 부분 변수가 있는 프롬프트 템플릿 정의
prompt = PromptTemplate(
    template="{season}에 일어나는 대표적인 지구과학 현상은 {phenomenon}입니다.",
    input_variables=["phenomenon"],  # 사용자 입력이 필요한 변수
    partial_variables={"season": get_current_season}  # 함수를 통해 동적으로 값을 생성하는 부분 변수
)

# 'phenomenon' 변수만 입력하여 현재 계절에 맞는 문장 생성
print(prompt.format(phenomenon="낙엽"))
~~~
## 마치며
다양한 프롬프트 방식이 존재하는데, 내가 사용하는 모델은 chatgpt api(IT, instruct-tuning 버전으로 명령어를 이해하는 생성답변을 생성하도록 파인튜닝됨)이기 때문에 system instruction을 주어서 많이 사용할 것 같다. 정확성을 위해 현재 상황에 대해 어떤게 나은지 직접 테스트 해봐야겠다. 다음으로는 chain을 구성하는 방법에 대해 공부할 예정이다.


```plaintext
💡참고   
https://wikidocs.net/book/14314   
https://wikidocs.net/book/14473   
https://python.langchain.com/v0.2/docs/introduction/ 
https://platform.openai.com/docs/guides/text-generation/quickstart?text-generation-quickstart-example=text
```