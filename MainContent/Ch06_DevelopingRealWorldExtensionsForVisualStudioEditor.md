---
sort: 6
---

# 비주얼 스튜디오 편집기용 리얼월드 확장 개발
지금까지는 개발자가 가장 자주 사용하는 Visual Studio의 핵심 부분인 코드 편집기를 확장하지 않았습니다. Visual Studio 편집기는 다양한 확장 지점 집합을 제공하며 대부분의 부분은 MEF(Managed Extensibility Framework)를 통해 확장할 수 있습니다. 사실, 편집기가 노출한 확장성 포인트가 너무 많아서 각각을 자세히 다루려면 책 한 권이 필요합니다. 다행스럽게도 Microsoft는 이미 이러한 확장성 지점과 기능을 매우 깊이 있게 문서화했습니다. 이 장에서는 Visual Studio 편집기의 확장성 포인트에 대해 논의하고 코드 분석, 수정 및 리팩토링을 위한 확장을 개발합니다. Visual Studio 편집기 확장을 시작하기 전에 편집기, 해당 하위 시스템, 기능 및 확장성 지점을 이해하는 것이 중요합니다. 다음 섹션에서는 편집기에 대해 논의한 다음 확장에 대해 자세히 설명합니다.

***

## <font color='dodgerblue' size="6">1) 비주얼 스튜디오 에디터</font>
Visual Studio 편집기는 아마도 Visual Studio에서 가장 자주 사용되는 구성 요소일 것입니다. 여기에서 개발자가 지원되는 언어 중 하나로 코드를 작성, 보기, 편집 및 디버그할 수 있습니다. 그림 6-1은 C# 언어용 Visual Studio 편집기와 그 여러 부분을 보여줍니다. 다른 부품에는 쉽게 참조하고 식별할 수 있도록 번호가 매겨져 있습니다. 번호가 매겨진 항목은 다음에 논의됩니다. 그것들은 다음과 같습니다:

![06_01_CodeEditor](image/06/06_01_CodeEditor.png)   
그림 6-01 Visual Studio 코드 편집기

  - **1. 코드 편집기**  
    문서가 편집되고 표시되는 핵심 편집기 영역입니다. IntelliSense의 핵심 기능도 이 지역에서 등장합니다. 이 영역을 마우스 오른쪽 버튼으로 클릭하면 이전 장에서 이미 보고 논의한 컨텍스트 메뉴가 표시됩니다.

  - **2. 프로젝트 드랍다운**  
    이 드롭다운은 파일이 열려 있는 활성 프로젝트를 표시합니다. Visual Studio에서 고아 파일을 열면 이 드롭다운에 기타 파일이 표시됩니다.

  - **3. 타입 드랍다운**  
    이 드롭다운에는 활성 문서에 정의된 모든 유형이 나열됩니다. 이 드롭다운은 선택한 값을 편집 중이거나 활성화된 유형, 즉 커서가 있는 위치로 표시합니다.

  - **4. 멤버 드랍다운**  
    선택또는 활성 타입의 모든 멤버(생성자, 필드, 속성, 메쏘드)가 나열.

  - **5. 윈도우 분할**  
        코드 편집기 창을 분할하기 위해.

  - **6. 스크롤바**  
        코드 창의 특정 섹션으로 쉽게 이동하고 창의 섹션을 보기 위해 사용합니다. 그림 6-1은 수직 스크롤바를 보여줍니다. 코드에 따라 편집기 하단에 가로 스크롤 막대가 표시될 수도 있습니다.

  - **7. 줄 번호**  
        파일에서 줄번호 표시

  - **8. 선택 마진**  
        이것은 라인 번호와 아웃라인 표시기 사이의 위치입니다. 코드를 수정할 때 해당 행은 코드 편집기의 왼쪽에 있는 색상으로 강조 표시됩니다. 선택 여백은 코드 변경 사항을 표시하는 데 사용되며 클릭 한 번으로 전체 코드 줄을 선택할 수 있습니다.

  - **9. 중괄호 완성**  
        편집기는 중괄호 완성 기능을 제공하고 해당 개구부를 강조 표시합니다.

  - **10. 표시기 여백**  
        이것은 편집기의 왼쪽에 있는 얇은 회색 영역입니다. 코드 줄에 대해 중단점과 책갈피가 표시되는 영역입니다.

  - **11. 확대 레벨**  
        이것은 코드 편집기를 확대하거나 축소하여 편집기에서 더 크거나 작은 글꼴 크기를 보는 데 사용할 수 있습니다.

이제 우리는 Visual Studio 편집기의 높은 수준의 개요를 보았습니다. 이제 편집기의 하위 시스템에 대해 논의할 시간입니다.

***

- ### A. 에디터 서브시스템
    Visual Studio 편집기는 사용자 인터페이스 계층과 편집기에 바인딩된 데이터(텍스트) 계층, 그리고 양쪽 계층을 분리해서 처리하는 다양한 모듈 또는 하위 시스템으로 구성되어진다.  
    그림 6-2는 Visual Studio 편집기의 상위 수준 하위 시스템 4개를 보여주고 있다. 당신이 웹개발을 해보았다면 MVC(모델 보기 컨트롤러) 디자인 패턴과 유사하게 생각하면 이러한 하위 시스템들을 쉽게 기억할 수 있다. 일반적으로 4개의 하위 시스템이 있다.

    ![06_02_EditorSubsystem](image/06/06_02_EditorSubsystem.png)   
    그림 6-02 Visual Studio 코드 편집기

    - **1. 텍스트 모델 서브시스템**  
        편집기에 표시할 데이터 모델을 나타낸다. 이 서브시스템은 편집기에 표시될 텍스트의 조작 및 투영을 담당한다. 이 하위 시스템은 다음 기능에 대한 유형을 제공한다.

        - 텍스트를 파일과 연결하고 파일 시스템에서 파일 읽기 및 쓰기를 관리하는 서비스.
        - 두 개체 시퀀스 간의 차이를 최소한의 차이을 찾는 차이점 서비스.
        - 여러 텍스트 버퍼의 텍스트를 결합하는 방법을 제공하는 프로젝션 시스템.
    
        그림 6-3은 이전 기능을 설명하는 유형이 포함된 네임스페이스를 보여줍니다.

        ![06_03_TextDifferProjection](image/06/06_03_TextDifferProjection.png)   
        그림 6-03 텍스트, 차이점, 프로젝션

        그림 6-3에서 우리가 참조하는 모든 클래스가 Microsoft.VisualStudio.Text.Data.dll에 있다는 것을 알 수 있다. 위에서 논의한 측면을 처리하는 세 가지 다른 네임스페이스가 있다. 이러한 네임스페이스를 확장하면 여러 유형이 포함되어 있음을 알 수 있다. 각각에 대해 논의하는 것은 이 책의 범위를 벗어난다.  
        
        Microsoft.VisualStudio.Text 네임스페이스에서 볼 수 있는 몇 가지 중요한 유형은 ITextChange, ITextChange2, ITextChange3, ITextEdit, ITextVersion, ITextVersion2, ITextSnapshot, IUndoEditTag, IDeletedTag 및 ITextBuffer 등이 있는데 텍스트 데이터를 조작, 정렬, 추적하는 메쏘드들을 보유한다. 그림 6-4는 텍스트 모델 하위 시스템을 구성하는 몇 가지 클래스를 보여준다.

        ![06_04_SomeTypesOfText](image/06/06_04_01_SomeTypesOfTextManipul.png)
        ![06_04_SomeTypesOfText](image/06/06_04_02_SomeTypesOfTextManipul.png)   
        
        이 장의 끝에 있는 "클래스 참조" 섹션은 텍스트 모델 하위 시스템의 중요한 구성원에 대한 높은 수준의 개요를 제공한다.

        이러한 모든 유형은 .NET 프레임워크 기본 클래스 및 MEF에 따라 다르다. 마찬가지로, Differencing 및 Projection 네임스페이스에도 다양한 유형이 있지만 여기서는 자세히 다루지 않을 것이다. 그러나 필요할 경우에는 유형의 세부 사항을 살펴 볼 것이다.

        Visual Studio에서 파일로 작업할 경우 파일의 종류를 알고 있다. C# 코드 파일 또는 XML 파일, JSON 파일 또는 csproj 파일, vb 코드 파일 등이 될 수 있다. 이를 식별하는 한 가지 방법은 파일 확장자이다. 확장자를 보면 파일 형식을 식별할 수 있다. 그러나 확장성 관점에서 새로운 사용자 지정 파일 형식(예: .verma 파일)을 추가한 후 Visual Studio에서 이를 지원해야 한다면 이 파일 형식에 대한 C# IntelliSense 지원은 어떻게 할수 있을까?

        Visual Studio는 이러한 시나리오를 처리할 수 있는 ContentType이라는 것을 정의한다. 콘텐츠 유형은 텍스트, 코드, 데이터, 바이너리 등과 같은 다양한 유형의 콘텐츠를 정의하는 기술이다. 또는 C#, JSON, XML 등과 같은 기술 유형이다. 많은 편집기 기능과 확장성 지점이 콘텐츠 유형에 연결됩니다. 예를 들어 편집기에서 C# 파일을 열면 상단에 프로젝트, 유형 및 멤버에 대한 3개의 드롭다운이 표시되지만 JSON, XML 또는 텍스트 파일을 열면 사라진다. 마찬가지로 구문 강조 표시, 색상 지정 및 IntelliSense도 이러한 모든 파일 형식에 대해 다르다. 콘텐츠마다 다른 처리가 필요하고 콘텐츠 유형을 기반으로 이러한 확장 기능을 정의하여 처리하기 때문이다.

        "클래스 참조" 섹션에는 문서 URL과 함께 콘텐츠 유형을 구성하는 중요한 유형이 요약되어 있다.

    - **2. 텍스트 뷰 서브시스템**  
        이 하위 시스템은 편집기에서 텍스트의 서식을 지정하고 표시하는 역할을 한다. MVC 비유의 관점에서 보면 텍스트 뷰는 편집기의 프레젠테이션 계층을 구성하는 보기 부분이다. 편집기에서 볼 수 있는 것은 WPF(Windows Presentation Foundation) 엘리멘트뿐이다. 이것은 본질적으로 텍스트를 표시하는 편집기의 UI 하위 시스템이다. 이 하위 시스템의 형식은 Microsoft.VisualStudio.Text.UI.dll 및 Microsoft.VisualStudio.Text.UI.Wpf.dll에 존재한다. WPF로 끝나는 어셈블리 이름에는 WPF 요소가 포함되고 다른 하나에는 플랫폼 독립적 요소가 포함된다. 따라서 이 시스템의 유형은 WPF와 플랫폼 독립의 두 계층으로 나뉩니다. 그림 6-5는 텍스트 보기 하위 시스템의 네임스페이스를 보여줍니다.
        
        ![06_05_NameSpaceTextview](image/06/06_05_NameSpaceTextview.png)   
        그림 6-05 텍스트뷰 서브시스템의 네임스페이스

        이 두 어셈블리 모두에서 몇 가지 네임스페이스가 공통인것에 주목하자. 하나는 플랫폼 독립적 구현을 포함하고 다른 하나는 WPF 특정 구현을 포함할 것으로 예상됩니다. 이 하위 시스템의 몇 가지 중요한 유형은 Microsoft.VisualStudio.Text.Editor 네임스페이스에 있는 ITextView 및 IWpfTextView이다. 또한 형식 지정, 삽입, 검색, 개요, 중괄호 완성, 태깅, 스크롤 등의 유형들이 있다. 
        
        "클래스 참조" 섹션에는 텍스트 보기 하위 시스템의 몇 가지 중요한 유형이 요약되어 있다.

    - **3. 분류(Classification) 서브시스템**  
        Visual Studio 편집기에서 C# 코드를 입력할 때 키워드, 주석, 기본 클래스, using 지시문 등을 서로 다른 색상을 할당하여 분리하는 훌륭한 작업을 수행한다. 이것은 텍스트를 다른 클래스로 분류하고 텍스트를 글꼴 속성에 매핑하는 편집기의 분류 서브 시스템때문에 가능한 것이다. 텍스트 범위에 마커를 추가하는 방법인 태깅 유형도 이 하위 시스템에서 정의된다. 이 하위 시스템의 유형은 그림 6-6과 같이 Microsoft.VisualStudio.Text.Logic.dll에 정의되어 있습니다.

        ![06_06_TypeOfClassification.png](image/06/06_06_TypeOfClassification.png)   
        그림 6-06 분류 서브시스템의 타입들
        
        분류 하위 시스템의 정보는 뷰 하위 시스템에서 텍스트 형식을 지정하고 분류 형식에서 매핑된 글꼴로 표시하는 데 사용됩니다.

    - **4. operation 서브시스템**  
        이름에서 알 수 있듯이 이 하위 시스템은 모든 편집기 작업, 명령 및 동작을 담당한다. 이 하위 시스템의 유형은 여러 어셈블리에 정의되어 있으며 그 중 일부는 앞에서 본 것입니다. 
        
    
    다음 섹션에서 편집기 기능과 확장성 포인트에 대해 논의해 보겠습니다.

- ### B. 에디터 기능들
    Visual Studio 편집기는 매우 풍부하고 다양한 기능을 제공합니다. 편집기 기능은 확장 가능하도록 훌륭하게 설계되었습니다. 이를 위해 각 기능에는 추상화 및 구현이 있습니다. 몇 가지 중요한 기능과 간략한 논의가 다음에 요약되어 있습니다.

    - **1. Tags**  
        텍스트 범위에 태그를 지정하거나 표시하는 데 사용됩니다. 태깅은 텍스트에 강조 표시, 색상 지정, 개요 또는 밑줄을 긋거나 그래픽 또는 팝업을 표시하여 시각적으로 표시됩니다. 예를 들어 구문 강조 표시는 분류자를 통해 수행되므로 일종의 태그입니다. 태깅을 사용하여 컴파일 오류와 같은 오류를 표시할 수도 있습니다. 태그 지정에 사용되는 몇 가지 유형은 다음과 같습니다.

        - **TextMarkerTag**
          텍스트 하이라이팅 용도 
        - **OutliningRegionText**
          아웃라인 표시 
        - **ErrorTag**
          에러 표시용. 편집기의 많은 기능들이 태그에 기반. 그림 6-7은 강조된 구불구불 에러 태그 표시

        ![06_07_ErrorTag](image/06/06_07_ErrorTag.png)   
        그림 6-07 에러 태그 작동

    - **2. 분류기(Classifier)**  
        분류기는 이름에서 알 수 있듯이 텍스트를 카테고리하거나 분류합니다. 분류기는 여러 분류 또는 범주가 있을 수 있으므로 IClassifier 인터페이스를 구현하여 콘텐츠 유형에 대해 정의됩니다. 그것들은 IClassificationType 인터페이스에 의해 정의됩니다. 분류기는 코드 텍스트를 주석, 키워드 또는 식별자로 분류할 수 있습니다.
        영어 알파벳을 모음, 자음 등으로 분류합니다. 분류기 형태의 인스턴스를 classification(분류)라고 한다. 그런 다음 다음 역할을 하는 분류기 집계기가 있습니다.
        분류기는 텍스트를 분류 세트로 나눕니다. 편집기의 텍스트 형식은 텍스트 분류를 기반으로 합니다. 텍스트는 키워드, 리터럴, 주석, 식별자 등으로 분류되며, 텍스트 보기 하위 시스템은 분류에 따라 텍스트의 서식을 지정하고 강조 표시하고 색상을 지정합니다. 그림 6-8은 C# 코드의 다른 부분을 표시하는 데 사용되는 다양한 색상을 보여줍니다. 이것은 분류를 통해 발생합니다.

        ![06_08_ToolTipAdornment](image/06/06_08_DiffClassifiColor.png)   
        그림 6-08 다른 분류와 다른 색깔

    - **3. Adornments**  
        장식이라는 단어의 문자적 의미는 꾸미기 또는 장식입니다. 마찬가지로 Visual Studio 편집기의 컨텍스트에서 텍스트의 글꼴 및 색상을 제외한 모든 텍스트 장식 또는 그래픽을 장식이라고 합니다. 오류 및 경고에 대한 코드 조각의 빨간색과 녹색 물결선은 장식의 좋은 예입니다. 툴팁은 또 다른 장식품입니다. WPF 기반의 편집기 UI에 표시되기 때문에 모든장식은 UIElement에서 파생되고 ITag를 구현해야 합니다. 그림 6-9는 편집기에서 string 키워드 위로 마우스를 가져갈 때 string에 대한 도구 설명 장식을 보여줍니다.

        ![06_09_ToolTipAdornment](image/06/06_09_ToolTipAdornment.png)   
        그림 6-09 툴팁 꾸미기

    - **4. Projection**  
        투영은 여러 텍스트 버퍼의 텍스트를 결합하고 다른 텍스트 버퍼를 구성하는 기술입니다. 따라서 투영을 사용하여 다른 버퍼의 텍스트를 결합하여 버퍼의 전체 텍스트를 표시하거나 텍스트의 일부만 표시하고 다른 부분은 숨길 수 있습니다. Visual Studio 편집기의 개요 기능을 사용할 때 코드 섹션을 축소하거나 확장할 수 있습니다. 이것은 텍스트 버퍼의 투영에 의해 달성됩니다. 그림 6-10은 투영을 통해 내부적으로 수행되는 축소된 코드 조각을 보여줍니다.

        ![06_10_CollapsedCodeBlock](image/06/06_10_CollapsedCodeBlock.png)   
        그림 6-10 코드 블록 축소
        
    - **5. 윤곽선(Outlining)**  
        이것은 편집기의 알려진 기능입니다. 편집기에서 코드 블록의 섹션을 확장하거나 축소할 수 있습니다. 아웃라이닝은 장식품과 마찬가지로 일종의 태그로 정의됩니다. OutliningRegionTag는 확장하거나 축소할 수 있는 텍스트 영역을 정의합니다. IOutliningManagerService 유형은 Visual Studio 편집기에서 코드 섹션을 확장하거나 축소하기 위해 ICollapsible 개체가 나타내는 다양한 텍스트 블록을 열거, 확장 또는 축소할 수 있는 IOutliningManager를 제공합니다. 이는 그림 6-11에 나와 있습니다. 강조 표시된 부분은 접을 수 있는 윤곽선입니다.
        
        ![06_11_OutlineVSEditor](image/06/06_11_OutlineVSEditor.png)   
        그림 6-11 비주얼 스튜디오 편집기에서 아웃라이닝

    - **6. Operations**  
        이름에서 알 수 있듯이 이것은 편집기의 상호 작용을 자동화하는 데 사용됩니다. 이전 버전의 Visual Studio는 편집기를 포함하여 Visual Studio에서 반복적인 작업을 자동화하는 데 선호되는 방식이었던 매크로를 지원했었다. 편집기와의 상호 작용을 자동화하기 위해 먼저 IEditorOperationsFactoryService를 가져와 텍스트 뷰의 작업에 액세스합니다. 그런 다음 이 개체를 사용하여 선택 항목을 수정하거나 필요에 따라 스크롤 막대 위치 등을 변경할 수 있습니다. 그림 6-12는 스크롤바, 분할 창, 캐럿 및 도구 설명이 작동하는 모습을 보여줍니다.

        ![06_12_ScrollSplitCaret](image/06/06_12_ScrollSplitCaret.png)   
        그림 6-12 스크롤바, 스플릿 윈도우, 카렛, 툴팁이 작동중(번호 순으로)

    - **7. IntelliSense**  
        Visual Studio 편집기에서 가장 많이 사용되는 기능을 살펴보면 IntelliSense가 확실히 상위권에 포함될 것입니다. 기본적으로 명령문 완성, 서명 도움말, 빠른 정보 및 전구 스타일 리팩토링을 지원하는 편집기의 상황에 맞는 지능형 센스쟁이이다. 명령문 완성 기능은 컨텍스트에서 적용 가능한 메서드 이름, API 또는 기타 마크업 요소에 대한 가능한 완성 목록을 제공합니다. IntelliSense는 사용자가 마침표(.) 또는 (Ctrl .)를 입력할 때 호출되며, 이는 잠재적인 완료 목록을 표시하는 완료 세션을 시작합니다. 사용자는 그 중 하나를 선택하거나 목록을 모두 닫을 수 있습니다. ICompletionBroker 유형은 지정된 세션에 대해 ICompletionSource 유형으로 계산된 CompletionSet 유형에 포함된 항목의 가능한 목록을 표시하는 ICompletionSession을 만들고 트리거하는 역할을 합니다. 그림 6-13은 자동 완성을 보여줍니다.
        
        ![06_13_Autocomplete](image/06/06_13_Autocomplete.png)   
        그림 6-13 자동완성

        그림 6-14는 Visual Studio의 전구 스타일 리팩토링을 보여줍니다. 이 장의 뒷부분에서 이러한 기능을 모두 구현하는 방법을 배웁니다.

        ![06_14_LightBulbRefactoring](image/06/06_14_LightBulbRefactoring.png)   
        그림 6-14 밝은 전구 형태의 리팩토링

        이러한 모든 기능과 그 이상은 온라인 Visual Studio 편집기 설명서에 매우 잘 설명되어 있으며 독자들이 시간을 할애하여 https에서 자세한 설명서를 읽을 것을 적극 권장합니다.  
        
        https://docs.microsoft.com/en-us/visualstudio/extensibility/inside-the-editor?view=vs-2019.


    다음으로 Visual Studio 편집기에서 제공하는 다양한 확장성 지점을 살펴보겠습니다.

## <font color='dodgerblue' size="6">2) 에디터 확장성</font>
Visual Studio 편집기는 확장성이 뛰어나며 편집기를 MEF(Managed Extensibility Framework) 구성 요소 부분으로 확장하는 데 사용할 수 있는 확장 지점을 제공합니다. 지난 섹션에서 논의한 편집기 기능은 모두 확장 가능합니다. 이러한 기능 중 일부의 경우 Visual Studio 확장성 템플릿이 직접 항목 템플릿을 제공하는 반면, 일부 기능의 경우 수동으로 클래스를 추가하고 올바른 인터페이스 및 클래스에서 파생하여 사용자 지정 코딩을 수행해야 합니다. Visual Studio는 분류자, 여백, 장식 및 뷰포트 장식을 위한 확장성 항목 템플릿을 제공합니다. 이것은 확장성 프로젝트에 항목을 추가하고 그림 6-15와 같이 새 항목 추가 대화 상자의 왼쪽 패널에서 확장성 ➤ 편집기를 선택하면 볼 수 있습니다.

![06_15_EditorExtTemplate](image/06/06_15_EditorExtTemplate.png)   
그림 6-15 에디터 확장성 아이템 템플릿

이러한 항목 템플릿 중 하나를 선택한 다음 확장을 빌드 및 디버그/배포하면 편집기 기능 확장을 시연하는 작업 확장이 있습니다. 이러한 항목 템플릿을 추가하면 코드 파일 추가, 참조 추가, vsixmanifest에 MefComponent 유형의 자산 노드 추가 등과 같이 백그라운드에서 많은 작업을 수행합니다.
그러나 요구 사항에 따라 기능을 사용자 지정하려면 코드를 수정해야 하지만 기능을 확장하고 이러한 템플릿을 연결하는 방법을 확인하는 것만으로도 좋은 시작입니다. 모든 독자가 이러한 각 템플릿을 사용해 보고 MEF 내보내기 및 가져오기가 유형에 어떻게 장식되고 모든 것이 연결되는지 확인하도록 권장합니다. 이러한 기능 외에도 템플릿이 제공되지 않는 다른 기능이 많이 있습니다. 이러한 확장성 포인트 각각을 다루려면 책 자체가 필요하므로 이 책의 범위를 벗어납니다. 하지만 좋은 소식은 이러한 모든 기능을 확장하는 것이 매우 잘 문서화되어 있으며
편집기 기능을 확장하는 프로세스는 필요에 따라 제공됩니다. 독자들에게 언어 및 편집기 서비스 확장에 대한 다음 Visual Studio 문서를 읽을 것을 적극 권장합니다.

https://docs.microsoft.com/en-us/visualstudio/extensibility/language-service-and-editor-extension-points?view=vs-2019.


다음 섹션에서는 C# 편집기에서 라이브 코드 진단 분석기를 실행하고 위반 시 빨간색 물결선을 표시하고 코드 수정을 제안하는 확장을 개발합니다.

- ### A. 코드 교정기능이 있는 진단 분석기
    라이브 진단 분석기가 필요한 이유에 대한 질문이 떠오를 수 있습니다. Visual Studio에는 수많은 코드 분석 및 진단 분석기가 내장되어 있으므로 진단 분석기를 이미 알고 사용했을 수 있습니다. 이전 장에서 확장 프로그램을 개발하는 동안 그림 6-16과 같이 기본 UI 스레드에서 코드를 실행해야 할 때마다 코드 아래에 구불구불한 자국이 있음을 기억하십시오.
    
    ![06_16_DiagnosticAnalyzer](image/06/06_16_DiagnosticAnalyzer.png)   
    그림 6-16 작동중인 진단분석기

    이것은 이제 우리가 사용한 VSIX 프로젝트 템플릿에 기본적으로 제공되는 분석기를 통해 수행됩니다. 물결선 위로 마우스를 가져가면 도구 팁과 전구가 보입니다. 이것은 본질적으로 코드 수정입니다. 전구 옆에 있는 아래쪽 마커를 클릭하면 문제를 해결하기 위한 제안이 표시됩니다. 제안을 클릭하면 문제가 해결됩니다. 제안은 또한 그림 6-17과 같이 문서, 프로젝트 또는 솔루션에서 논의 중인 메서드의 코드 수정 미리 보기 또는 이러한 모든 경우의 수정을 표시하도록 제안합니다.

    ![06_17_CodeFix](image/06/06_17_CodeFix.png)   
    그림 6-17 코드 교정

    미리 보기 변경 사항을 클릭하면 코드 수정 사항이 추가된 후 코드가 어떻게 보일지 보여주는 멋진 대화 상자가 표시됩니다. 코드 변경 사항에 만족하면 적용을 클릭하거나 취소 버튼을 클릭하여 취소할 수 있습니다. 이는 그림 6-18에 나와 있습니다.

    ![06_18_PreviewCodeFix](image/06/06_18_PreviewCodeFix.png)   
    그림 6-18 코드 교정 변경사항 미리보기

    솔루션 탐색기에서 확장 프로젝트(VSIX 템플릿에서 생성)를 연 다음 참조, 분석기 노드를 차례로 확장하면 프로젝트에서 세 개의 분석기를 볼 수 있습니다. 세 가지 분석기는 다음과 같습니다.

    - Microsoft.CodeAnalysis.CSharp.BannedApiAnalyzers
    - Microsoft.VisualStudio.SDK.Analyzers
    - Microsoft.VisualStudio.Threading.Analyzers

    그림 6-19는 확장된 규칙 집합과 함께 VSIX 템플릿에 포함된 세 가지 분석기를 표시합니다.

    ![06_19_CodeAnalyzersVsix](image/06/06_19_CodeAnalyzersVsix.png)   
    그림 6-19 VSIX프로젝트에서 코드 분석기

    분석기가 제공하는 실시간 피드백을 활용하여 코딩하는 동안 우리 개발자는 그렇지 않으면 포착되지 않거나 코드 검토에서 걸리거나 프로덕션에서 더 악화될 수 있는 많은 문제를 피할 수 있습니다. 따라서 진단 분석기는 특히 대규모 팀에서 모범 사례, 코딩 표준 및 팀 지침을 보다 쉽게 채택할 수 있습니다. 각 규칙의 심각도는 규칙 작성자가 결정할 수 있습니다.

    이전의 FxCop과 같은 다른 정적 코드 분석 도구와 달리 분석기의 또 다른 장점은 피드백을 제공하기 위해 생성하기 위해 코드를 컴파일하고 어셈블할 필요가 없다는 것입니다. 한편으로는 오래된 정적 분석 도구가 제공하는 피드백이 늦거나 지연되었기 때문에 여러 번 팀은 FxCop을 포기하라는 힘든 요청을 받아들여야 했습니다.

    분석기는 NuGet 패키지 또는 Visual Studio 확장으로 배포할 수 있습니다. 따라서 분석기를 쉽게 찾을 수 있습니다. NuGet 갤러리 또는 Visual Studio Marketplace에서 검색하고 설치할 수 있습니다. 사실, 모든 Visual Studio 프로젝트의 속성에는 코드 분석 탭이 있습니다. 이를 사용하여 그림 6-19와 같이 소스 코드 분석기 또는 바이너리 분석기를 구성할 수 있습니다. 그림 6-20에서 번호가 표시되면 #1은 프로젝트 속성 페이지의 코드 분석 탭을 나타냅니다. 프로젝트 속성 페이지는 솔루션 탐색기에서 프로젝트를 마우스 오른쪽 버튼으로 클릭한 다음 상황에 맞는 메뉴에서 속성을 클릭하여 볼 수 있습니다(또는 프로젝트가 선택된 상태에서 Alt Enter 키를 누름). 숫자 2는 소스 분석기를 표시합니다. 이 페이지는 권장되는 분석기 패키지도 표시하고 설치된 버전을 표시하는 UI와 새 분석기를 제거/설치하는 옵션을 제공합니다. 또한 빌드 시 분석기를 실행하고 라이브 분석 시 실행하는 옵션도 표시합니다. 소스 코드 분석기의 이점을 보여주는 링크도 있습니다. 한 번 이상 읽어야 합니다. 3번은 더 이상 사용되지 않는 바이너리 분석기 섹션을 보여줍니다. 또한 빌드 시 분석기를 실행하는 옵션을 제공합니다. 그리고 #4는 필요에 따라 규칙을 선택/선택 해제할 수 있는 활성 규칙 집합을 구성하는 방법을 표시합니다. 그림 6-21은 규칙 집합을 구성하는 화면을 표시합니다. (여기에는 수정할 수 없는 Microsoft 규칙 집합이 표시되어 있으므로 수정 후 이 규칙 집합을 다른 이름으로 저장하고 프로젝트에서 참조할 수 있습니다).

    ![06_20_ConfigeAnalyzers](image/06/06_20_ConfigeAnalyzers.png)   
    그림 6-20 프로젝트 등록정보에서 분석기 구성
    
    ![06_21_ConfigureRuleSet](image/06/06_21_ConfigureRuleSet.png)   
    그림 6-21 분석기 룰셋 구성

- ### B. 코드 픽스와 진단 코드 분석기 만들기
    Visual Studio에는 진단 분석기를 더 쉽게 개발할 수 있는 프로젝트 템플릿이 있습니다. 따라서 다음으로 코드 교정이 포함된 진단 코드 분석기를 개발할 것입니다. 

    async await는 C#에서 개발된 대부분의 최신 웹, 데스크톱 또는 플랫폼 간 앱에서 매우 널리 사용됩니다. .NET Core에는 SynchronizationContext, .NET이 없지만 전체 프레임워크에는 포함되어 있으며 .NET Core 또는 .NET 전체 프레임워크 클래스 라이브러리가 .NET 프레임워크 애플리케이션에서 사용되는 경우 SynchronizationContext는 성능을 결정하는 데 중요한 역할을 하며 또한 응용 프로그램이 교착 상태에 빠질 수 있는지 여부. 어떻게? WPF/Windows Forms 앱은 소유 스레드만 UI 컨트롤을 수정하거나 해당 값을 업데이트할 수 있도록 요구하는 것과 같이 다양한 유형의 애플리케이션에는 서로 다른 스레딩 모델이 있습니다. 다른 스레드에서 UI 컨트롤을 업데이트하려고 하면 예외가 발생합니다.

    마찬가지로 ASP.NET/ASP.NET Core에는 이러한 제한이 없습니다. 이것은 SynchronizationContext라는 클래스에 의해 컴파일러 수준에서 처리됩니다. 메서드를 비동기로 표시하면 컴파일러는 이 메서드 코드를 여러 번 호출할 수 있고 다른 위치에서 실행을 시작할 수 있는 상태 머신을 구현하는 형식으로 변환합니다. 이 비동기 메서드에서 await 문이 발생하면 컴파일러는 기본적으로 ContinueWith 대리자 또는 연속에서 메서드 코드의 나머지 부분을 연결하고 결과 코드는 컨텍스트를 인식합니다. 따라서 SynchronizationContext가 있는 경우 await 식은 SynchronizationContext 개체에 대한 참조를 가져오고 이 개체를 사용하여 연속을 호출합니다. API가 비동기적으로 호출되고 결과 데이터가 UI 컨트롤에 바인딩되므로 PF UI 레이어에는 문제가 없습니다. UI 컨트롤은 소유자 스레드에서만 업데이트할 수 있으므로 SynchronizationContext는 API 호출이 반환될 때 호출 스레드가 작업 결과를 반환하도록 합니다. 이 상황에서는 이것이 편리하지만 호출 스레드로 다시 마샬링하는 추가 비용이 있습니다.

    ASP.NET/ASP.NET Core와 같은 대부분의 경우 비동기 메서드를 호출하는 스레드가 전체 연속 코드도 완료하는지 여부는 신경 쓰지 않습니다. 따라서 스레드를 마샬링하거나 컨텍스트 인식을 유지하기 위해 SynchronizationContext가 필요하지 않습니다. 이를 위해 동일한 컨텍스트에서 계속하도록 컴파일러에 지시하는 데 사용할 수 있는 ConfigureAwait라는 메서드가 있습니다. 또한 UI 쓰레드가 무언가를 하다가 차단되는 경우(.Wait() 또는 .Result를 이용한 잘못된 코드)가 발생하고 이 시간 동안 UI 쓰레드에서 호출된 비동기 API가 반환됩니다. 이제 SynchronizationContext는 캡처된 컨텍스트, 즉 다른 차단 호출에서 차단된 UI 스레드에서 실행을 반환하려고 시도하여 교착 상태가 발생합니다. 따라서 모든 라이브러리 메서드는 await 문과 함께 .ConfigureAwait(false)를 사용해야 하므로 교착 상태 또는 성능 저하로 이어지는 상황을 피할 수 있습니다.

    await 문에 ConfigureAwait(false)가 있는지 여부를 확인하는 간단한 진단 분석기를 작성할 것입니다. 그렇지 않으면 물결선이 표시됩니다. 또한 ConfigureAwait(false)에 대한 호출을 추가하여 이 문제를 해결하기 위한 간단한 코드 수정을 제공합니다. 이 규칙은 이제 분석기에 내장되어 있지만 학습 및 데모 목적으로 개발할 때 이 분석기를 계속 사용합니다.

    코드 분석기를 개발하는 동안 다양한 용어와 전문 용어를 접하게 되므로 표 6-1과 같이 이러한 용어를 빠르게 살펴보겠습니다.

    표 6-1. 일반적으로 사용되는 코드 분석 용어
    ```
    멤버                            설명
    ------------------------------  -------------------------------------------------------------
    Diagnostic                      발생한 위치에 따른 컴파일러 오류, 경고 또는 코드 패턴 위반들.
    Analyzer                        진단을 보고하는 클래스입니다. DiagnosticAnalyzer에서 파생된 타입의 인스턴스
    Code fixer                      컴파일러 또는 분석기 진단을 위한 코드 교정을 제공하는 클래스. 
                                    CodeFixProvider에서 파생된 타입의 인스턴스.   
    Code Refactoring                소스 코드 리팩토링을 제공하는 유형. CodeRefactoringProvider에서 파생된 유형의 인스턴스.
    Code action                     코드 교정 또는 코드 리팩토링 작업을 나타냅니다.
    Equivalence Key                 코드 작업을 등록할 때마다 이것이 실제로 작동하는 것을 볼 수 있습니다. 등록된 모든 코드 작업의
                                    등가 클래스를 나타내는 문자열 값. 둘 이상의 코드 작업은 동일한 등가 키 값을 공유하고 동일한
                                    코드 수정 프로그램 또는 리팩토링에 의해 생성되는 경우 동등한 것으로 처리됩니다.
    FixAll provider                 FixAll 발생 코드 수정을 제공하는 클래스입니다. FixAllProvider에서 파생된 유형의 인스턴스.
    FixAll occurrences code fix     FixAllProvider.GetFixAsync에서 반환한 코드 작업으로, 지정된 FixAllScope에 대해 해당 코드 수정
                                    프로그램에서 수정한 진단의 전체 또는 여러 항목을 수정합니다.
    ```

    - **1. 프로젝트 셋업**  
        진단 분석기 코딩을 시작해 보자. Visual Studio 2019를 사용하여 진단 분석기를 개발하려면 다음 단계를 완료해야 합니다.
        
        - a  
            Visual Studio 2019를 열고 새 프로젝트를 만듭니다. 그림 6-22와 같이 프로젝트 템플릿 "Analyzer with Code Fix"를 검색합니다.

            ![06_22_AnalyzerWithCodeFix](image/06/06_22_AnalyzerWithCodeFix.png)   
            그림 6-22 Analyzer With Code Fix

        - b  
            프로젝트에 의미 있는 이름을 지정한 다음 그림 6-23과 같이 만들기 버튼을 클릭합니다.

            ![06_23_ConfigureAwaitAnalyzerProject](image/06/06_23_ConfigureAwaitAnalyzerProject.png)   
            그림 6-23 ConfigureAwaitAnalyzer 프로젝트

            그러면 그림 6-24와 같이 3개의 프로젝트가 추가됩니다.
            
            ![06_24_AddedProjects](image/06/06_24_AddedProjects.png)   
            그림 6-24 추가된 프로젝트들


            - ConfigureAwaitAnalyzer.csproj  
                클래스 라이브러리 프로젝트입니다. 여기에서 진단 분석기 클래스와 코드 수정 클래스를 작성합니다. 이름이 Analyzer 및 CodeFixProvider로 끝나므로 쉽게 식별할 수 있습니다. 이 프로젝트에는 현지화를 위한 리소스 파일과 분석기를 설치하거나 제거하기 위한 PowerShell 스크립트도 포함되어 있습니다.

            - ConfigureAwaitAnalyzer.Test  
                테스트 프로젝트입니다. 이 프로젝트는 진단 분석기 및 코드 수정을 테스트하는 데 사용할 수 있습니다.

            - ConfigureAwaitAnalyzer.Vsix  
                Visual Studio용 클래스 라이브러리 확장을 패키징하는 VSIX 프로젝트입니다. 이 프로젝트에는 ConfigureAwaitAnalyzer 클래스 라이브러리 프로젝트에 대한 프로젝트 참조가 있으며 vsixmanifest 파일이 포함되어 있습니다.

                이 vsixmanifest 파일은 MEF 구성 요소가 확인될 때 코드 분석기를 연결하는 자산 섹션의 두 가지 중요한 자산 노드를 정의합니다. 이러한 노드의 코드는 다음과 같습니다.

                ```xml
                <Assets>
                    <Asset Type="Microsoft.VisualStudio.MefComponent" d:Source="Project" 
                        d:ProjectName="ConfigureAwaitAnalyzer" Path="|ConfigureAwaitAnalyzer|"/>
                    <Asset Type="Microsoft.VisualStudio.Analyzer" d:Source="Project" 
                        d:ProjectName="ConfigureAwaitAnalyzer" Path="|ConfigureAwaitAnalyzer|"/>
                </Assets>
                ```

                ![06_25_AssetInVsixManifest](image/06/06_25_AssetInVsixManifest.png)   
                그림 6-25 VSIX 매니페스트 파일의 자산정보들

                테스트 주도 개발(TDD)을 옹호하는 사람들에게는 단위 테스트를 작성하는 것이 시작하는 방법일 수 있습니다. 그러나 나는 다른 학파에 속하므로 먼저 분석기를 작성한 다음 단위 테스트를 작성합니다. 템플릿에서 생성된 코드는 이미 작동하는 코드 분석기입니다. 그러나 유형 이름에 소문자가 포함되어 있는지 여부만 감지합니다. 그렇다면 물결선이 표시됩니다. 진단을 작성하기 위해 ConfigureAwaitAnalyzerAnalyzer.cs 파일을 편집하기 전에 먼저 기존 코드를 이해해야 합니다. 또한 이름에 Analyzer가 반복되어 있으므로 파일 이름을 변경해야 합니다. 그림 6-26은 진단 분석기의 클래스 다이어그램을 보여줍니다. 클래스 다이어그램에서 다음을 볼 수 있습니다.

                - analyzer 클래스는 클래스이며 DiagnosticAnalyzer라는 다른 추상 클래스에서 파생됨.
                - 기본 클래스 DiagnosticAnalyzer에는 System.Object에 의해 노출되는 멤버 외에 관심할 만한 두 멤버가 있다.
                    - SupportedDiagnostics
                    - Initialize

                ![06_26_DiagnosticAnalyzer](image/06/06_26_DiagnosticAnalyzer.png)   
                그림 6-26 진단분석기
                    
    - **2. SupportedDiagnostics**  
        ImmutableArray<DiagnosticDescriptor> 유형의 추상 읽기 전용 속성. 파생 클래스는 분석기가 지원하는 진단 기능들을 제공한다. DiagnosticDescriptor는 진단에 대한 설명을 제공하는 Microsoft.CodeAnalysis 네임스페이스에 정의된 봉인된 클래스이다. 진단은 Microsoft.CodeAnalysis 네임스페이스에 정의된 또 다른 추상 클래스로, 컴파일러 오류 또는 경고와 같은 진단을 발생한 위치와 함께 나타냅니다. 여러 오버로드가 있는 진단을 만드는 정적 메서드를 제공합니다. 이러한 유형은 모두 앞의 클래스 다이어그램에 나와 있습니다. 이러한 유형의 멤버는 이해하기 쉽고 나중에 코딩에서 볼 수 있듯이 매우 직관적입니다. 그러나 완료를 위해 DiagnosticDescriptor 및 Diagnostic 클래스의 중요한 멤버는 "클래스 참조" 섹션에 문서화되어 있습니다.
    - **3. Initialize**  
        이것은 분석 컨텍스트에서 작업을 등록하기 위해 세션 시작 시 한 번 호출되는 추상 메서드입니다. 여기에는 이전 클래스 다이어그램에서 추상 클래스로 표시되는 AnalysisContext 유형의 컨텍스트라는 매개변수가 있습니다. 여기에서 분석기에 대한 진단 작업을 등록합니다. Analyzer 초기화는 AnalysisContext를 사용하여 컴파일 시작 또는 끝, 코드 문서 구문 분석 완료 또는 코드 또는 기호의 의미론적 분석 완료 시 실행할 수 있는 작업을 등록할 수 있습니다.

        AnalysisContext 추상 클래스의 중요한 멤버는 "클래스 참조" 섹션에 설명되어 있습니다.  
        더 자세한 문서는 https://docs.microsoft.com/en-us/dotnet/api/microsoft.codeanalysis.diagnostics.analysiscontext?view=roslyn-dotnet

        다른 유형은 LocalizableResourceString으로 추상 클래스 LocalizableString에서 파생되며 이 리소스 문자열이 문화권에 따라 다르게 형식화될 수 있으므로 지역화에 사용됩니다. 분석기를 현지화하지 않으려면 일반 문자열도 선택할 수 있습니다. 클래스에 장식할 수 있는 DiagnosticAnalyzerAttribute라는 특성이 있습니다. 이 속성을 유형에 배치하면 진단 분석기로 간주됩니다.
        
        진단 분석기 코드와 관련된 다양한 유형과 해당 멤버를 알았으므로 이제 ConfigureAwait 메서드가 호출되지 않은 await 문을 살펴보기 위해 진단 분석기의 코드를 조정해 보겠습니다.

    - **4. 진단 분석기 코딩**  
        그렇게 하려면 Roslyn(.NET Compiler 플랫폼) 기술을 사용해야 합니다. Roslyn은 코드 작업을 위한 두 가지 유형의 모델, 즉 SyntaxTree 및 SemanticModel을 제공합니다.

        - SyntaxTree는 코드를 구문 분석하여 생성할 수 있으며 코드의 개체 모델 표현과 같습니다. 변경할 수 없으므로 수정할 수 없습니다. SyntaxTree를 수정하면 새로운 SyntaxTree 객체가 생성됩니다. 컴파일 정보가 없습니다. 예를 들어 콘솔과 같은 코드를 텍스트 식별자 또는 리터럴로 이해합니다.

        - 컴파일 정보는 SematicModel이라는 다른 모델에 의해 제공됩니다. SematicModel도 변경 불가능하고 코드를 이해합니다(예: Console을 System.Console과 같은 유형으로 이해).

        ConfigureAwait가 호출되었는지 여부를 식별하려면 SyntaxTree 또는 구문 분석으로 충분합니다. 그렇게 하려면 먼저 wait 문을 식별한 다음 ConfigureAwait 메서드를 호출하는지 여부를 확인해야 합니다. 삶을 더 쉽게 만들기 위해 Visual Studio는 Syntax Visualizer라는 도구 창을 제공했습니다. 이 창을 열려면 상단 메뉴 모음으로 이동한 다음 보기 ➤ 다른 창 ➤ 구문 시각화 도우미를 클릭합니다. 그러면 Visual Studio 측면 중 하나에 고정할 수 있는 도구 창이 열립니다. 코드를 편집하는 동안 내용을 더 쉽게 볼 수 있도록 왼쪽에 고정했습니다. 편집기에서 코드를 입력하거나 편집기에서 행을 클릭하면 그림 6-27과 같이 Syntax Visualizer 창에서 노드의 SyntaxTree에 해당 노드가 표시됩니다.

        ![06_27_SyntaxVisualizer](image/06/06_27_SyntaxVisualizer.png)   
        그림 6-27 Syntax Visualizer
        
        ConfigureAwait에 대한 호출 없이 await 문이 있는 편집기에서 간단한 비동기 메서드를 작성했습니다. 이 진단이 이 프로젝트의 분석기 중 하나에 있으므로 이미 물결선이 있습니다. 당분간 무시하거나 규칙 세트에서 이 규칙을 비활성화하십시오. 이 방법의 방향 그래프는 그림 6-28에 나와 있습니다. 보시다시피, 이 정보에서 전체 코드를 다시 재구성할 수 있을 정도로 매우 상세하므로 ConfigureAwait 없이 wait 문을 찾는 방법을 확실히 찾을 수 있습니다. 편집기에서 await 문을 클릭하고 Syntax Visualizer 창의 Syntax 트리에서 강조 표시된 노드를 찾으면 await 문을 찾기 위해 Syntax 트리에서 볼 노드를 찾을 수 있습니다. 이 경우 구문 트리에서 Await 문이 AwaitExpression으로 표시된다는 것을 알았습니다. ConfigureAwait 호출을 추가하면 SimpleMemberAccessExpression 내부에 IdentifierName 토큰으로 표시됩니다.
        
        ![06_28_DirectGraphAsync](image/06/06_28_DirectGraphAsync.png)   
        그림 6-28 async 메쏘드의 직접 그래프

        이 정보를 가지고 코드를 수정할 준비가 되었다.

        - a  
            Diagnostic 클래스에서는 먼저 DiagnosticId를 제공한다. Microsoft에서 사용하는 CA로 시작하는 ID를 사용해서는 안 되므로 RV001(Rishabh Verma의 약자 RV)을 사용할 것입니다. 원하는 다른 문자열을 선택할 수 있습니다. 다음으로 그림 6-29와 같이 적절한 제목, 메시지 형식 및 설명을 추가하도록 리소스 파일을 수정합니다.
            
            ![06_29_ResFileWithDiagTitle](image/06/06_29_ResFileWithDiagTitle.png)   
            그림 6-29 진단 제목, 설명, 메시지 포맷으로 수정된 리소스 파일

        - b  
            다른 변경사항들은 그림 6-30에서 강조되었다

            ![06_30_ChangesConfigureAwait](image/06/06_30_ChangesConfigureAwait.png)   
            그림 6-30 ConfigureAwait 분석기용 코드 변경점

            범주를 Reliability로 변경하고 Initialize 메서드를 수정하여 컨텍스트에 SyntaxNode 작업을 등록했습니다. 코드는 AwaitExpression을 만날 때마다 AnalyzeAwaitStatements 메서드를 호출합니다. AnalyzeAwaitStatements의 코드는 다음과 같습니다.
    
            ```cs
            private void AnalyzeAwaitStatements(SyntaxNodeAnalysisContext obj)
            {
                if (!obj.Node.DescendantNodes().OfType<IdentifierNameSyntax>().Any(
                        j => j.Identifier.ValueText.Equals("ConfigureAwait", StringComparison.OrdinalIgnoreCase)
                    ))
                {
                    var containingMethod = obj.Node.Ancestors().OfType<MethodDeclarationSyntax>().FirstOrDefault();
                    var diagnostic = Diagnostic.Create(Rule, obj.Node.GetLocation(), containingMethod.Identifier.ValueText);
                    obj.ReportDiagnostic(diagnostic);
                }
            }
            ```

            코드는 이해하기 쉽습니다. 우리 메서드의 매개변수는 SyntaxAnalysisContext 유형이며, 이는 구조체이고 컨텍스트에서 SyntaxNode, Compilation, ContainingSymbol, SemanticModel 및 Options 속성을 가져오는 속성을 포함합니다. 또한 진단을 보고하는 데 사용할 수 있는 ReportDiagnostic이라는 메서드가 있습니다. AwaitExpression이 발생했을 때 작업을 등록했으므로 항상 메서드의 매개 변수로 AwaitExpression을 얻습니다. ConfigureAwait 코드가 SimpleMemberAccessExpression 내부에 IdentifierName으로 표시되는 것을 이미 앞에서 보았으므로 AwaitExpression의 자손 노드를 찾고 ConfigureAwait라는 Identifier를 검색합니다. 발견되면 문제가 없습니다. 그렇지 않으면 진단 개체를 만들고 컨텍스트 개체에서 ReportDiagnostic 메서드를 호출하고 진단 개체를 매개 변수로 전달합니다.
    - **5. 진단 분석기 실행**  
    - **6. 코드 픽스 만들기**  
    - **7. 확장도구 테스트**  
    - **8. 확장도구 배포**  

## <font color='dodgerblue' size="6">3) 코드 리팩토링 확장도구</font>
- ### A. 확장도구 코딩
- ### B. 리팩토링 테스팅

## <font color='dodgerblue' size="6">4) 인텔리센스</font>

## <font color='dodgerblue' size="6">5) 인텔리코드</font>

## <font color='dodgerblue' size="6">6) 요약</font>

## <font color='dodgerblue' size="6">7) 클래스 레퍼런스</font>


    