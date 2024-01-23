> 이 문서는 해당 앱과 관련 없는 안드로이드 테스트 관련 내용을 포함한다.

## 자동 테스트

테스트는 개발자가 작성한 코드가 올바르게 작동하는지 확인하는 코드이다. 자동 테스트는 코드를 테스트하는 주체가 소프트웨어가 되는 것이다.

코드베이스를 확장하려면 새 코드를 추가해야 하고, 새 코드를 추가하려면 기존 기능을 테스트해야 하는데, 이는 기존 테스트가 있는 경우에만 가능하다.
앱의 규모가 커지면 개발자가 수동으로 테스트를 할 때 자동 테스트보다 더욱 많은 노력이 필요하게 된다. 따라서 앱을 확장하면서 테스트를 작성나가는 것이 중요하다.

자동 테스트의 유형은 두 가지다.

### 1. 로컬 테스트

- 함수, 클래스, 속성 등 소수의 코드를 직접 테스트하여 제대로 작동하는지 확인하는 자동 테스트의 유형이다.
- 워크 스테이션에서 실행된다.
- 기기나 에뮬레이터 없이 개발 환경에서 실행된다.
- 컴퓨터 리소스의 오버헤드가 낮아 제한된 리소스에서 빠르게 실행할 수 있다.

#### 테스트 경로 구성
1. 테스트에 사용할 메서드의 공개 형식 지정자를 확인한다. `private`이면 안된다.
2. 테스트에 사용할 메서드 앞줄에 `@VisibleForTesting` 주석을 추가한다. 이 주석은 메서드가 공개되지만 테스트 목적으로만 공개된다고 표시하는 것이다.
3. Project의 `src` 디렉토리 밑에 `test/java` 디렉토리를 새로 만든다.
4. `test/java` 디렉토리에는 앱 코드가 있는 `main`디렉토리와 동일한 패키지 구조가 있어야 한다.
5. 테스트를 작성할 코틀린 파일을 지금까지 만든 패키지 안에 만든다. 

#### 테스트 작성
1. **테스트의 내용과 예상 결과를 명확하게 설명하는 이름**을 가진 메서드를 작성한다.
2. 메서드에 `@Test` 주석을 단다. 컴파일러는 이 주석을 보고 메서드가 테스트 메서드임을 인지한다.
3. 테스트는 지정된 입력의 예상 출력을 엄격하게 확인한다. 즉, 미리 정의된 입력을 메서드의 인수로 넣어서 메서드가 제대로 동작하는지 확인한다.
4. 테스트는 특정 조건이 충족되었는지 확인하는 `Assertion`으로 끝난다.
   ```kotlin
   import org.junit.Assert.assertEquals
   import org.junit.Test
   import java.text.NumberFormat

   class TipCalculatorTests {
     @Test
     fun calculateTip_20PercentNoRoundup() {
         val amount = 10.00
         val tipPercent = 20.00
         val expectedTip = NumberFormat.getCurrencyInstance().format(2)
         val actualTip = calculateTip(amount = amount, tipPercent = tipPercent, false)
         assertEquals(expectedTip, actualTip)
     }
   }
   ```

<br>

### 2. 계측 테스트

- 앱이나 앱의 일부를 실행하고 사용자 상호작용을 시뮬레이션하여 앱이 적절하게 반응했는지 확인하는 자동 테스트의 유형이다.
- 실제 기기 또는 에뮬레이터에서 실행된다.
- 계측 테스트을 실행하면 실제 일반 Android 앱처럼 APK 패키지로 빌드된다.

#### 테스트 경로 구성
1. Project의 `src` 디렉토리 밑에 `androidTest/java` 디렉토리를 새로 만든다.
2. `test/java` 디렉토리에는 앱 코드가 있는 `main`디렉토리와 동일한 패키지 구조가 있어야 한다.
3. 테스트를 작성할 코틀린 파일을 지금까지 만든 패키지 안에 만든다. 

#### 테스트 작성
1. `createComposeRule()`의 결과로 만들어진 Rule 변수를 만들고 `@Rule` 주석을 추가한다.
2. **테스트의 내용과 예상 결과를 명확하게 설명하는 이름**을 가진 메서드를 작성하고 `@Test` 주석을 붙인다.
3. 메서드 본문에서 `composeTestRule.setContent()`을 호출한다. `composeTestRule`의 UI 콘텐츠가 설정된다.
4. `MainActivity`에서 `theme`과 함께 컴포저블을 호출하는 형식을 똑같이 `setContent()` 내부에 작성한다.
5. UI 구성요소는 `composeTestRule`을 통해 노드로 접근할 수 있다. 주로 `onNodeWithText`와 같은 메서드로 특징을 통해 UI 요소를 불러온다.
6. `performTextInput()`과 같은 메서드를 통해 값들을 UI 요소에게 전달할 수 있다. 마치 사용자 상호작용 같은 기능이다.
7. 여러 메서드들을 실행하고 최종 결과를 어셜션을 통해 예상 결과와 비교한다.
   ```kotlin
   import java.text.NumberFormat

   @Test
   fun calculate_20_percent_tip() {
       composeTestRule.setContent {
           TipTimeTheme {
               Surface (modifier = Modifier.fillMaxSize()){
                     TipTimeLayout()
               }
           }
       }
       composeTestRule.onNodeWithText("Bill Amount")
          .performTextInput("10")
       composeTestRule.onNodeWithText("Tip Percentage").performTextInput("20")
       val expectedTip = NumberFormat.getCurrencyInstance().format(2)
       composeTestRule.onNodeWithText("Tip Amount: $expectedTip").assertExists(
            "No node with this text was found."
       )
    }
    ```





>[!Note]
> 컴파일러는 `androidTest` 디렉터리에서 `@Test` 주석이 달린 메서드를 계측 테스트라고 인지하고, `test` 디렉터리에선 로컬 테스트라고 인지한다.
