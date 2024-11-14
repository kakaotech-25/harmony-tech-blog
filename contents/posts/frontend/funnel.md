---
title: 코드 재사용성을 높이기 위한 Funnel 구조
date: "2024-11-14"
writer: 리안
tags:
  - React
  - Next
  - Typescript
  - Funnel
  - 코드 재사용
  - 리안
previewImage: funnel.png
---

## 코드 재사용성을 높여야 한다.

가상 면접 서비스를 구현하기 위해 피그마를 통해 페이지를 간단하게 디자인 해보았다. 페이지를 다 그려보고 나니 겹치는 디자인이 상당히 많았다.  
특히 면접 모드, 면접 유형, 면접 방식, 직무 선택, 자기소개서 입력, 정보 확인 페이지는 내부의 디자인이 유사했고 각 페이지를 이전과 다음으로 자주 이동하기에 새로운 페이지로 연결하면 로딩이 짧지만 조금씩 생기는 것도 하나의 문제였다.  
이 문제를 해결하고 코드를 효율적으로 관리하기 위해 방법을 알아보던 도중 토스에서 funnel 방식으로 구현한 예시를 보여주는 영상을 보게 되어 이를 프로젝트에 적용하게 되었다. Funnel 구조를 통해 각 단계를 개별 컴포넌트로 나누고, 상태를 관리하며 필요한 조건에 따라 전환할 수 있도록 하였다.

## Funnel 구조란??

최근 기업들이 페이지를 구성할 때 온보딩이나 예약 과 같은 사용자 흐름에서 funnel 구조를 많이 활용한다.  
funnel 구조는 각 단계의 페이지를 관리하면서 특정 조건에 따라 적절한 페이지를 보여주기 위한 구조로 특히 단계별 진행이 필요한 페이지에서 가장 효율적으로 사용할 수 있다.
위에서 언급한 토스의 경우에는 가입과정과 같은 사용자 설정 단계를 funnel 방식으로 구현하여 조건에 따른 페이지 전환을 자동화하며 코드 재사용성을 높였다.

## 구현 과정

단계별 페이지를 컴포넌트로 작성하고 현재 단계에 맞는 특정 컴포넌트를 렌더링하는 방식으로 설계하였다.

### Funnel 컴포넌트 구현

```ts
"use client"
import React, {
  Children,
  isValidElement,
  ReactElement,
  ReactNode,
  useEffect,
} from "react"

export type NonEmptyArray<T> = [T, ...T[]]
export type StepsType = Readonly<NonEmptyArray<string>>

export interface FunnelProps<Steps extends StepsType> {
  steps: Steps
  step: Steps[number]
  children:
    | Array<ReactElement<StepProps<Steps>>>
    | ReactElement<StepProps<Steps>>
}

export const Funnel = <Steps extends StepsType>({
  steps,
  step,
  children,
}: FunnelProps<Steps>) => {
  const validChildren = Children.toArray(children)
    .filter(isValidElement)
    .filter(i =>
      steps.includes((i.props as Partial<StepProps<Steps>>).name ?? "")
    ) as Array<ReactElement<StepProps<Steps>>>

  const targetStep = validChildren.find(child => child.props.name === step)

  return <>{targetStep}</>
}

export interface StepProps<Steps extends StepsType> {
  name: Steps[number]
  onEnter?: () => void
  children: ReactNode
}

export const Step = <Steps extends StepsType>({
  onEnter,
  children,
}: StepProps<Steps>) => {
  useEffect(() => {
    onEnter?.()
  }, [onEnter])

  return <>{children}</>
}

Funnel.Step = Step
```

Funnel 컴포넌트는 steps와 step 속성을 받아서 현재 단계인 step에 해당하는 컴포넌트로 렌더링한다.  
이 과정을 통해 각페이지의 전환을 깔끔하게 관리할 수 있다.

### SetupFunnel 구현

```ts
"use client"
import { useRouter } from "next/navigation"
import TypeStep from "./step/type-step"
import MethodStep from "./step/method-step"
import JobStep from "./step/job-step"
import CoverLetterStep from "./step/cover-letter-step"
import CheckInfoStep from "./step/check-info-step"
import { Funnel } from "../funnel"

type StepType = "type" | "method" | "job" | "cover-letter" | "check-info"

interface SetupFunnelProps {
  step: StepType
  setStep: (step: StepType) => void
  interviewMode: "GENERAL" | "REAL"
}

const SetupFunnel = ({ step, setStep, interviewMode }: SetupFunnelProps) => {
  const router = useRouter()

  const steps =
    interviewMode === "GENERAL"
      ? (["type", "method", "job", "check-info"] as const)
      : (["type", "method", "job", "cover-letter", "check-info"] as const)

  const handleSubmit = (interviewId: number) => {
    router.push(`/interview/ongoing/${interviewId}`)
  }

  return (
    <Funnel steps={steps} step={step}>
      <Funnel.Step name="type">
        <TypeStep
          onPrev={() => {
            setStep("type")
            router.push("/interview")
          }}
          onNext={() => {
            setStep("method")
          }}
        />
      </Funnel.Step>
      <Funnel.Step name="method">
        <MethodStep
          onPrev={() => {
            setStep("type")
          }}
          onNext={() => {
            setStep("job")
          }}
        />
      </Funnel.Step>
      <Funnel.Step name="job">
        <JobStep
          onPrev={() => {
            setStep("method")
          }}
          onNext={() => {
            setStep(interviewMode === "GENERAL" ? "check-info" : "cover-letter")
          }}
        />
      </Funnel.Step>
      <Funnel.Step name="cover-letter">
        <CoverLetterStep
          onPrev={() => {
            setStep("job")
          }}
          onNext={() => {
            setStep("check-info")
          }}
        />
      </Funnel.Step>
      <Funnel.Step name="check-info">
        <CheckInfoStep
          onPrev={() => {
            setStep(interviewMode === "GENERAL" ? "job" : "cover-letter")
          }}
          onNext={() => {
            setStep("check-info")
          }}
          onSubmit={handleSubmit}
        />
      </Funnel.Step>
    </Funnel>
  )
}

export default SetupFunnel
```

SetupFunnel을 통해 여러 단계의 면접 설정 페이지를 하나의 흐름으로 이어줄 수 있다.  
단계마다 다른 컴포넌트를 연결하고 각 단계에서의 동작과 전환을 관리한다.  
특히 면접왕김개발 서비스에서는 모의면접과 실전면접에 따라 자기소개서 페이지의 차이가 있기때문에 모의면접 상태를 mock으로 받아와서 자기소개서 입력 페이지를 생략할 수 있도록 step를 관리하고 있다. 이런 구조를 통해 설정에 맞게 필요한 단계를 추가할 때 편하게 스텝을 추가할 수 있다.

### InterviewSetupPage 구현

```ts
"use client"
import { Suspense, useEffect, useState } from "react"
import { useRouter, useSearchParams } from "next/navigation"
import SetupFunnel from "@/components/interview/setup-funnel"
import SetupProgressBar from "@/components/interview/setup-progress-bar"

type StepType = "type" | "method" | "job" | "cover-letter" | "check-info"

const getStepLabel = (step: StepType) => {
  const labels: Record<StepType, string> = {
    type: "면접 유형 선택",
    method: "면접 방식 선택",
    job: "직무 선택",
    "cover-letter": "자기소개서 입력",
    "check-info": "입력 정보 확인",
  }
  return labels[step]
}

const InterviewSetupPage = () => {
  const router = useRouter()
  const searchParams = useSearchParams()
  const interviewMode =
    (searchParams.get("type") as "GENERAL" | "REAL") || "REAL"
  const [currentStep, setCurrentStep] = useState<StepType>("type")

  const steps: StepType[] =
    interviewMode === "GENERAL"
      ? ["type", "method", "job", "check-info"]
      : ["type", "method", "job", "cover-letter", "check-info"]

  useEffect(() => {
    const step = searchParams.get("step") as StepType
    if (!step) {
      router.replace(`/interview/setup?type=${interviewMode}&step=type`)
    } else {
      setCurrentStep(step)
    }
  }, [searchParams, router, interviewMode])

  const updateStep = (newStep: StepType) => {
    setCurrentStep(newStep)
    router.push(`/interview/setup?type=${interviewMode}&step=${newStep}`)
  }

  return (
    <>
      <div className="text-center text-[50px] font-bold mb-16">
        {getStepLabel(currentStep)}
      </div>
      <div className="pb-10 px-8 mb-8">
        <SetupProgressBar steps={steps} currentStep={currentStep} />
      </div>
      <div className="flex flex-col items-center gap-8">
        <SetupFunnel
          step={currentStep}
          setStep={updateStep}
          interviewMode={interviewMode}
        />
      </div>
    </>
  )
}

export default function PageWrapper() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <InterviewSetupPage />
    </Suspense>
  )
}
```

여기서는 실제로 단계별 설정을 표시하는 페이지 내부 구성을 구현하였다.
프로그레스바와 현재 단계를 보여주고 사용자가 현재 어느 단계에서 정보를 입력하고 있는지 확인할 수 있다.  
currentStep 상태를 활용해 각 단계를 전환하고 있으며 사용자가 이전이나 다음 단계로 넘어갈 때 setCurrentStep을 호출하여 새로운 단계로 업데이트하고 URL을 변경한다.

### 단계별 컴포넌트 구현

각 step 마다 약간씩은 다른 페이지 구성을 가지고 있기 때문에 각각 다른 파일로 구현하였지만 파일이 여러 개인 관계로 한 페이지를 예시로 소개하고자 한다.

```ts
"use client"
import { useState } from "react"
import NavButtons from "./nav-button"
import SettingBtn from "@/components/settingbtn"
import useInterviewStore, { InterviewType } from "@/stores/useInterviewStore"

const TypeStep = ({ onPrev, onNext }: StepProps) => {
  const [selectedType, setSelectedType] = useState<InterviewType>()
  const { updateInterviewField } = useInterviewStore()

  const handleClick = () => {
    if (selectedType) {
      updateInterviewField("interviewType", selectedType)
    }
  }

  return (
    <>
      <div className="flex gap-16 justify-center">
        <SettingBtn
          label="기술면접"
          description="관심 직무를 기반으로 기술적 지식, 문제 해결 능력, 실무 적용 역량을 평가하는 면접"
          selected={selectedType === "TECHNICAL"}
          onClick={() => setSelectedType("TECHNICAL")}
        />
        <SettingBtn
          label="인성면접"
          description="지원자의 성격, 가치관, 대인관계 능력 등을 질문을 통해 직접 확인하고 평가하는 면접"
          selected={selectedType === "PERSONAL"}
          onClick={() => setSelectedType("PERSONAL")}
        />
      </div>

      <NavButtons
        onPrev={onPrev}
        onNext={() => {
          if (selectedType) {
            onNext()
            handleClick()
          }
        }}
        prevButtonText="이전"
        nextButtonText="다음"
        disabled={!selectedType}
      />
    </>
  )
}

export default TypeStep
```

실전면접인지 인성면접인지 선택하는 페이지다.
selectedType이 있을 때만 다음버튼이 활성화되도록 하여 다음 단계로 넘어가도록 하였다.

## Funnel 구조의 장점

첫번째로 코드 재사용성이 증가했다. 각 단계가 컴포넌트로 관리되어 재사용성이 높아졌기 때문에 반복적으로 같은 코드를 작성할 필요가 없어졌다.  
두번째로는 페이지 로딩속도가 줄어들었다. step에 따라 넘겨주는 형태이기 때문에 실제로 페이지에 들어가보면 funnel로 관리되고 있지 않은 모드선택 -> 유형선택 전환속도와 funnel로 관리되고 있는 유형선택 -> 방식선택 전환속도를 비교해보면 funnel이 빠르다는 것을 확인할 수 있다.

## 모행서비스로의 도입!

모행서비스에서도 회원가입 단계에서 이런식으로 프로그레스바가 포함되어있고 단계로 관리되는 흐름이 있지만 funnel 구조로 구현되지 않아 step이 아닌 다른 URL과 다른 렌더링 과정으로 구현되어 있어서 페이지 전환시 매끄럽지 않은 부분이 있다. 프로그레스바 또한 이미지로 넣어져서 부자연스러운 디자인으로 구현되어 있다. funnel 구조는 step으로 관리되고 있어서 프로그레스의 전환 또한 반복되는 작업없이 관리할 수 있으며 자연스럽게 페이지간 이동 또한 할 수 있다.  
funnel 구조로 페이지를 개발해보면서 모행서비스 리팩토링 과정에서 적용해보고 싶다는 생각을 하게 되었다. 리팩토링 기간이 시작되면 한번 적용해보고 싶은 마음이다.
