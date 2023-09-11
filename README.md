## 测试说明

- Query
- 断言
- User Actions(Fire Events and user event)
- Mock API
- Redux test

## Query

- Query 是 Testing Library 提供的方法用来获取页面上的元素

- Query 中 getBy,queryBy 和 findBy 的区别

  - 如果要判断一个元素不存在,并进行断言.这时候如果使用 getBy 就会导致测试报错.使用 queryBy 便能正常的进行.
  - findBy 通常用于异步元素.例如

    ```
    describe('App', () => {
    test('renders App component', async () => {
            render(<App />);
            expect(await screen.findByText(/Signed in as/)).toBeInTheDocument();
        });
    });
    ```

  - 断言多个元素,可以使用多元素查询方法 getAllBy / queryAllBy / findAllBy

- 一些常用的 Query 方法

  - ByRole
  - 方法： getByLabelText / queryByRole / findByRole / getAllByRole / queryAllByRole / findAllByRole

    根据元素的 Role 来获取元素。例如

    ![image](https://github.com/taojack/wholesaleware-web/blob/testRules/docs/IMG/getByRoleExample.PNG)

    PS：有些元素已有默认的 role, 例如`<button />`的 role 就是 button,具体请参考 https://www.w3.org/TR/html-aria/#docconformance

    PS：role 可以添加 hidden, selected, checked...等属性来过滤，具体请参考 https://testing-library.com/docs/queries/byrole

  - ByLabelText
  - 方法： getByLabelText / queryByLabelText / getAllByLabelText / queryAllByLabelText / findByLabelText / findAllByLabelText

    根据 Label 元素的 InnerHTML value 来获取元素。例如

    ![image](https://github.com/taojack/wholesaleware-web/blob/testRules/docs/IMG/getByLabelTextExample.PNG)

    PS：可以使用`getByRole('textbox', { name: 'Username' }) `来代替。

    PS：可以使用 selector 来标注具体的元素。请参考 https://testing-library.com/docs/queries/bylabeltext

  - ByPlaceholderText
  - 方法： getByPlaceholderText/queryByPlaceholderText/getAllByPlaceholderText/queryAllByPlaceholderText/findByPlaceholderText/findAllByPlaceholderText

    根据元素的 placeholder 来获取元素。例如

    ![image](https://github.com/taojack/wholesaleware-web/blob/testRules/docs/IMG/getByPlaceholderTextExample.PNG)

  - ByText
  - 方法： getByText / queryByText / getAllByText / queryAllByText / findByText / findAllByText

    根据元素的 InnerHTML value 来获取元素。例如

    ![image](https://github.com/taojack/wholesaleware-web/blob/testRules/docs/IMG/getTextExample.PNG)

  - ByDisplayValue
  - 方法： getByDisplayValue/queryByDisplayValue/getAllByDisplayValue/queryAllByDisplayValue/findByDisplayValue/findAllByDisplayValue

    根据元素的 value 来获取元素

    ![image](https://github.com/taojack/wholesaleware-web/blob/testRules/docs/IMG/getByDisplayValueExample.PNG)

  - ByAltText
  - 方法： getByAltText / queryByAltText / getAllByAltText / queryAllByAltText / findByAltText/ findAllByAltText

    根据元素的 alt 来获取元素

    ![image](https://github.com/taojack/wholesaleware-web/blob/testRules/docs/IMG/getByAltTextExample.PNG)

  - ByTitle
  - 方法： getByTitle / queryByTitle / getAllByTitle / queryAllByTitle / findByTitle / findAllByTitle

    根据元素的 title 来获取元素

    ![image](https://github.com/taojack/wholesaleware-web/blob/testRules/docs/IMG/getByTitleExample.PNG)

  - ByTestId
  - 方法： getByTestId / queryByTestId / getAllByTestId / queryAllByTestId / findByTestId / findAllByTestId

    根据元素的 id 来获取元素

    ![image](https://github.com/taojack/wholesaleware-web/blob/testRules/docs/IMG/getByIdExample.PNG)

- 自定义的 Query 方法

  - getById
  - 方法使用： getById(domContainer, 'idName')
    [实现参考](https://testing-library.com/docs/dom-testing-library/api-custom-queries/)

    - 根据元素的 id 属性 来获取元素
    - 可用于获取 form 生成的表单 Input 项

    ```
      <div><input type="checkbox" id="autoSyncProductsCheckbox" /></div>

      const autoSyncProductsCheckbox = getById(
        document.body,
        'autoSyncProducts',
      ) as HTMLInputElement;
    ```

    - 或者 ant Select 点击后下拉后，获取关联的下拉列表部分的 dom

    ```
      <span class="ant-select-selection-search">
        <input id="userId" role="combobox" aria-haspopup="listbox" aria-autocomplete="list" aria-controls="userId_list" value="" aria-expanded="false">
      </span>

      const selectInput = getById(
        document.body,
        'userId',
      ) as HTMLInputElement;
      // mouseDown Select input to open select option list
      await act(async () => {
        fireEvent.mouseDown(selectInput);
      });

    const selectListBox = getById(
      document.body,
      selectInput.getAttribute('aria-controls') as string,
    )?.nextSibling as HTMLElement;
    ```

## 断言

- toBeEmpty

  用来检查一个元素是否为空

  ```
  <span data-testid="not-empty"><span data-testid="empty"></span></span>

  expect(getByTestId('empty')).toBeEmpty()
  expect(getByTestId('not-empty')).not.toBeEmpty()
  ```

- toBeDisabled

  用来检查一个元素是否是 disable。以下元素可以设置 disable 属性，button, input, select, textarea, optgroup, option, fieldset。
  例如：

  ```
  <button data-testid="button" type="submit" disabled>submit</button>
  <fieldset disabled><input type="text" data-testid="input" /></fieldset>
  <a href="..." disabled>link</a>

  expect(getByTestId('button')).toBeDisabled()
  expect(getByTestId('input')).toBeDisabled()
  expect(getByText('link')).not.toBeDisabled()
  ```

- toBeEnabled

  用来检查一个元素是否非 disable。
  代码中用 `.not.toBeDisabled()` 来表示

- toBeEmptyDOMElement

  用来检查一个元素是否有空的内容。这个方法会忽略 comment 但是不会忽略空格。例如：

  ```
  <span data-testid="not-empty"><span data-testid="empty"></span></span>
  <span data-testid="with-whitespace"> </span>
  <span data-testid="with-comment"><!-- comment --></span>

  expect(getByTestId('empty')).toBeEmptyDOMElement()
  expect(getByTestId('not-empty')).not.toBeEmptyDOMElement()
  expect(getByTestId('with-whitespace')).not.toBeEmptyDOMElement()
  ```

- toBeInTheDocument

  用来检查一个元素是否存在于 document 中，例如：

  ```
  <span data-testid="html-element"><span>Html Element</span></span>
  <svg data-testid="svg-element"></svg>

  expect(
  getByTestId(document.documentElement, 'html-element'),
  ).toBeInTheDocument()
  expect(getByTestId(document.documentElement, 'svg-element')).toBeInTheDocument()
  expect(
  queryByTestId(document.documentElement, 'does-not-exist'),
  ).not.toBeInTheDocument()
  ```

- toBeInvalid

  用来检查一个元素是否是无效的（如果元素的 aria-invalid 的值是 false 或者 checkValidity 返回 false，此 element 无效，具体请参考https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-invalid_attribute)

  例子：

  ```
  <input data-testid="no-aria-invalid" />
  <input data-testid="aria-invalid" aria-invalid />
  <input data-testid="aria-invalid-value" aria-invalid="true" />
  <input data-testid="aria-invalid-false" aria-invalid="false" />

  <form data-testid="valid-form">
  <input />
  </form>

  <form data-testid="invalid-form">
    <input required />
  </form>

  expect(getByTestId('no-aria-invalid')).not.toBeInvalid()
  expect(getByTestId('aria-invalid')).toBeInvalid()
  expect(getByTestId('aria-invalid-value')).toBeInvalid()
  expect(getByTestId('aria-invalid-false')).not.toBeInvalid()

  expect(getByTestId('valid-form')).not.toBeInvalid()
  expect(getByTestId('invalid-form')).toBeInvalid()

  ```

- toBeRequired

  用来检查一个元素是否存是 required，例如：

  ```
  <input data-testid="required-input" required />
  <input data-testid="aria-required-input" aria-required="true" />

  expect(getByTestId('required-input')).toBeRequired()
  expect(getByTestId('aria-required-input')).toBeRequired()
  ```

- toBeValid

  用来检查一个元素是否有效

  ```
  <input data-testid="no-aria-invalid" />
  <input data-testid="aria-invalid" aria-invalid />

  expect(getByTestId('no-aria-invalid')).toBeValid()
  expect(getByTestId('aria-invalid')).not.toBeValid()
  ```

- toBeVisible

  用来检查一个元素是否对用户可见。
  以下情况用户不可见：

  - 元素不在 document 中
  - display 设置为 none
  - visibility 设置为 hidden 或者 collapse
  - opacity 设置为 0
  - 父元素不可见
  - 设置 hidden 属性

  例子：

  ```
  <div data-testid="zero-opacity" style="opacity: 0">Zero Opacity Example</div>
  <div data-testid="visibility-hidden" style="visibility: hidden">
  Visibility Hidden Example
  </div>
  <div data-testid="display-none" style="display: none">Display None Example</div>
  <div style="opacity: 0">
  <span data-testid="hidden-parent">Hidden Parent Example</span>
  </div>
  <div data-testid="visible">Visible Example</div>
  <div data-testid="hidden-attribute" hidden>Hidden Attribute Example</div>

  expect(getByText('Zero Opacity Example')).not.toBeVisible()
  expect(getByText('Visibility Hidden Example')).not.toBeVisible()
  expect(getByText('Display None Example')).not.toBeVisible()
  expect(getByText('Hidden Parent Example')).not.toBeVisible()
  expect(getByText('Visible Example')).toBeVisible()
  expect(getByText('Hidden Attribute Example')).not.toBeVisible()
  ```

- toContainElement

  用来检查一个元素是否是另一个元素的父元素。例如：

  ```
  <span data-testid="ancestor"><span data-testid="descendant"></span></span>

  const ancestor = getByTestId('ancestor')
  const descendant = getByTestId('descendant')
  const nonExistantElement = getByTestId('does-not-exist')

  expect(ancestor).toContainElement(descendant)
  expect(descendant).not.toContainElement(ancestor)
  expect(ancestor).not.toContainElement(nonExistantElement)

  ```

- toContainHTML

  用来检查一个元素是否包含一个 html 元素字符串（此 html 元素字符串要可见并且完整）。例如：

  ```
  <span data-testid="parent"><span data-testid="child"></span></span>

  expect(getByTestId('parent')).toContainHTML('<span data-testid="child"></span>')
  expect(getByTestId('parent')).toContainHTML('<span data-testid="child" />')

  ```

- toHaveAccessibleDescription

  用来检查一个元素是否包含 accessible description。例如：

  ```
  <a data-testid="link" aria-label="Home page" title="A link to start over">Start</a>

  expect(getByTestId('link')).toHaveAccessibleDescription()
  expect(getByTestId('link')).toHaveAccessibleDescription('A link to start over')
  expect(getByTestId('link')).not.toHaveAccessibleDescription('Home page')

  ```

- toHaveAttribute

  用来检查一个元素是否包含相应的属性。例如：

  ```
  <button data-testid="ok-button" type="submit" disabled>ok</button>

  const button = getByTestId('ok-button')

  expect(button).toHaveAttribute('disabled')
  expect(button).toHaveAttribute('type', 'submit')
  expect(button).not.toHaveAttribute('type', 'button')

  ```

- toHaveClass

  用来检查一个元素是否包含相应的 class 属性。例如：

  ```
  <button data-testid="delete-button" class="btn extra btn-danger">
  Delete item
  </button>

  expect(deleteButton).toHaveClass('extra')
  expect(deleteButton).toHaveClass('btn-danger btn')
  expect(deleteButton).not.toHaveClass('btn-link')
  ```

- toHaveFocus

  用来检查一个元素是否被 focus。例如：

  ```
  <div><input type="text" data-testid="element-to-focus" /></div>

  const input = getByTestId('element-to-focus')

  input.focus()
  expect(input).toHaveFocus()

  input.blur()
  expect(input).not.toHaveFocus()
  ```

- toHaveFormValues

  用来检查一个 form 或者 fieldset 元素对于每一个 name 都有相应的 control 并且有相应的 value。

  - input 的各种 type 及 select 等请参考https://github.com/testing-library/jest-dom#toHaveFormValues

    ```
    <form data-testid="login-form">
    <input type="text" name="username" value="jane.doe" />
    <input type="password" name="password" value="12345678" />
    <input type="checkbox" name="rememberMe" checked />
    <button type="submit">Sign in</button>
    </form>

    expect(getByTestId('login-form')).toHaveFormValues({
      username: 'jane.doe',
      rememberMe: true,
    })
    ```

- toHaveStyle

  用来检查一个元素是否有相应的 css 属性。例如：

  ```
  <button
    data-testid="delete-button"
    style="display: none; background-color: red"
  >
  Delete item
  </button>

  const button = getByTestId('delete-button')

  expect(button).toHaveStyle('display: none')
  expect(button).toHaveStyle(`
    background-color: red;
    display: none;
  `)

  ```

- toHaveTextContent

  用来检查一个元素是否有 text content。例如：

  ```
  <span data-testid="text-content">Text Content</span>

  expect(element).toHaveTextContent('Content')
  expect(element).toHaveTextContent(/^Text Content$/)

  ```

- toHaveValue

  用来检查一个元素是否有 value。

  例如：

  ```
  <input type="text" value="text" data-testid="input-text" />
  <select multiple data-testid="select-number">
    <option value="first">First Value</option>
    <option value="second" selected>Second Value</option>
    <option value="third" selected>Third Value</option>
  </select>


  const textInput = getByTestId('input-text')
  const selectInput = getByTestId('select-number')

  expect(textInput).toHaveValue('text')
  expect(selectInput).not.toHaveValue(['second', 'third'])
  ```

  PS： `<input type="checkbox">`和`<input type="radio">`使用 toBeChecked 或者 toHaveFormValues 来检查

- toHaveDisplayValue

  用来检查一个 Form 的元素的最终 display value（用户会看到的 value）。例如：

  ```
  <label for="input-example">First name</label>
  <input type="text" id="input-example" value="Luca" />

  <label for="textarea-example">Description</label>
  <textarea id="textarea-example">An example description here.</textarea>

  <label for="single-select-example">Fruit</label>
  <select id="single-select-example">
    <option value="">Select a fruit...</option>
    <option value="banana">Banana</option>
  </select>

  <label for="multiple-select-example">Fruits</label>
  <select id="multiple-select-example" multiple>

    <option value="">Select a fruit...</option>
    <option value="banana" selected>Banana</option>
    <option value="ananas">Ananas</option>
    <option value="avocado" selected>Avocado</option>
  </select>


  const input = screen.getByLabelText('First name')
  const textarea = screen.getByLabelText('Description')
  const selectSingle = screen.getByLabelText('Fruit')
  const selectMultiple = screen.getByLabelText('Fruits')

  expect(input).toHaveDisplayValue('Luca')
  expect(input).toHaveDisplayValue(/Luc/)
  expect(textarea).toHaveDisplayValue('An example description here.')
  expect(textarea).toHaveDisplayValue(/example/)
  expect(selectSingle).toHaveDisplayValue('Select a fruit...')
  expect(selectSingle).toHaveDisplayValue(/Select/)
  expect(selectMultiple).toHaveDisplayValue([/Avocado/, 'Banana'])

  ```

- toBeChecked

  用来检查一个元素是否被 checked， 此方法用来检查 checkbox, radio 或者`<input type="checkbox">`。例如：

  ```
  <input type="checkbox" checked data-testid="input-checkbox-checked" />
  <input type="radio" checked value="foo" data-testid="input-radio-checked" />

  const inputCheckboxChecked = getByTestId('input-checkbox-checked')
  const inputRadioChecked = getByTestId('input-radio-checked')

  expect(inputCheckboxChecked).toBeChecked()
  expect(inputRadioChecked).toBeChecked()
  ```

## User Actions

- Firing Event 和 user Event 的区别

  user Event 依赖于 Firing Event。可以认为 user Event 使用 Firing Event 做了一系列的 actions。
  一个很好的例子是当用户使用 Firing Event 点击 Button 的时候，button 不会被 focused，但是使用 user Event，Button 会被 focused。
  原因就是 Firing Event 只模拟了点击行为，而 user Event 不但模仿了点击还模仿了 focused。
  所以 user Event 能够更好的模拟用户行为，推荐使用 user Event。

  备注： ant design 个别情况需要使用 firing event 来模拟，例如 datapicker 需要先点击一下等。。。

- user Event

  - click

    根据元素的 click()模拟 click 事件，没有 side-effect。例如：

    ```
    test('click', () => {
    render(
      <div>
        <label htmlFor="checkbox">Check</label>
        <input id="checkbox" type="checkbox" />
      </div>,
    )

      userEvent.click(screen.getByText('Check'))
      expect(screen.getByLabelText('Check')).toBeChecked()
    })
    ```

    同样可以使用 ctrlClick/shiftClick， 比如：

    ```
    userEvent.click(elem, {ctrlKey: true, shiftKey: true})
    ```

    备注： click 触发之前会触发 hover 事件，如果想禁止请使用 skipHover

  - dblClick

    模拟双击事件，没有 side-effect。例如：

    ```
    test('double click', () => {
      const onChange = jest.fn()
      render(<input type="checkbox" onChange={onChange} />)
      const checkbox = screen.getByRole('checkbox')
      userEvent.dblClick(checkbox)
      expect(onChange).toHaveBeenCalledTimes(2)
      expect(checkbox).not.toBeChecked()
    })
    ```

  - type

    模拟在`<input>`或`<textarea>`中写入 text， 例如：

    ```
    test('type', () => {
      render(<textarea />)

      userEvent.type(screen.getByRole('textbox'), 'Hello,{enter}World!')
      expect(screen.getByRole('textbox')).toHaveValue('Hello,\nWorld!')
    })
    ```

    - 一些特殊的操作及输入时间请参照https://testing-library.com/docs/ecosystem-user-event/#typeelement-text-options

  - keyboard

    模拟键盘时间，这个事件和 type 很相似，但是 type 时间没有任何的 click 事件和 change 事件，例如

    ```
    userEvent.keyboard('foo') // translates to: f, o, o
    userEvent.keyboard('{Shift}{f}{o}{o}') // translates to: Shift, f, o, o
    userEvent.keyboard('[ShiftLeft][KeyF][KeyO][KeyO]') // translates to: Shift, f, o, o
    userEvent.keyboard('{shift}{ctrl/}a{/shift}') // translates to: Shift(down), Control(down+up), a, Shift(up)
    ```

    - '{'和'['为特殊字符，可以输入两次来引用，例如：

      ```
      userEvent.keyboard('{{a[[') // translates to: {, a, [
      ```

    - 可以使用>和/来模拟按住某个键再点击另一个，例如：

      ```
      userEvent.keyboard('{Shift>}A{/Shift}') // translates to: 按住shift, A, 松开Shift
      ```

  - upload

    模拟文件上传事件。例如：

    ```
    test('upload file', () => {
      const file = new File(['hello'], 'hello.png', {type: 'image/png'})

      render(
        <div>
          <label htmlFor="file-uploader">Upload file:</label>
          <input id="file-uploader" type="file" />
        </div>,
      )
      const input = screen.getByLabelText(/upload file/i)
      userEvent.upload(input, file)

      expect(input.files[0]).toStrictEqual(file)
      expect(input.files.item(0)).toStrictEqual(file)
      expect(input.files).toHaveLength(1)
    })
    ```

    -对于多个文件的上传请使用 multiple 属性，并且 upload 的第二个参数应该是个 Array。例如：

    ```
    test('upload multiple files', () => {
      const files = [
        new File(['hello'], 'hello.png', {type: 'image/png'}),
        new File(['there'], 'there.png', {type: 'image/png'}),
      ]

      render(
        <div>
          <label htmlFor="file-uploader">Upload file:</label>
          <input id="file-uploader" type="file" multiple />
        </div>,
      )
      const input = screen.getByLabelText(/upload file/i)
      userEvent.upload(input, files)

      expect(input.files).toHaveLength(2)
      expect(input.files[0]).toStrictEqual(files[0])
      expect(input.files[1]).toStrictEqual(files[1])
    })
    ```

  - clear

    选择`<input>`或`<textarea>`中的值并删除，例如：

    ```
    test('clear', () => {
      render(<textarea value="Hello, World!" />)

      userEvent.clear(screen.getByRole('textbox'))
      expect(screen.getByRole('textbox')).toHaveAttribute('value', '')
    })
    ```

  - selectOptions

    模拟`<select>`或`<select multiple>`事件

    - 单选

      ```
      test('selectOptions', () => {
        render(
          <select data-testid="listbox">
            <option value="1">A</option>
            <option value="2">B</option>
            <option value="3">C</option>
          </select>,
        );

        userEvent.selectOptions(screen.getByTestId('listbox'), ['1']);

        expect(screen.getByRole('option', { name: 'A' }).selected).toBe(true);
        expect(screen.getByRole('option', { name: 'B' }).selected).toBe(false);
      });
      ```

    - 多选

      ```
        test('selectOptions', () => {
          render(
            <select multiple>
              <option value="1">A</option>
              <option value="2">B</option>
              <option value="3">C</option>
            </select>,
          )

          userEvent.selectOptions(screen.getByRole('listbox'), ['1', '3'])

          expect(screen.getByRole('option', {name: 'A'}).selected).toBe(true)
          expect(screen.getByRole('option', {name: 'B'}).selected).toBe(false)
          expect(screen.getByRole('option', {name: 'C'}).selected).toBe(true)
        })
      ```

  - deselectOptions

    模拟`<select multiple>`事件中取消选择

    ```
      test('deselectOptions', () => {
        render(
          <select multiple>
            <option value="1">A</option>
            <option value="2">B</option>
            <option value="3">C</option>
          </select>,
        )

        userEvent.selectOptions(screen.getByRole('listbox'), '2')
        expect(screen.getByText('B').selected).toBe(true)
        userEvent.deselectOptions(screen.getByRole('listbox'), '2')
        expect(screen.getByText('B').selected).toBe(false)
        // can do multiple at once as well:
        // userEvent.deselectOptions(screen.getByRole('listbox'), ['1', '2'])
      })
    ```

  - tab

    模拟更改选项卡的事件(tab 事件)，例如

    ```
      it('should cycle elements in document tab order', () => {
        render(
          <div>
            <input data-testid="element" type="checkbox" />
            <input data-testid="element" type="radio" />
            <input data-testid="element" type="number" />
          </div>,
        )

        const [checkbox, radio, number] = screen.getAllByTestId('element')

        expect(document.body).toHaveFocus()

        userEvent.tab()

        expect(checkbox).toHaveFocus()

        userEvent.tab()

        expect(radio).toHaveFocus()

        userEvent.tab()

        expect(number).toHaveFocus()

        userEvent.tab()

        // cycle goes back to the body element
        expect(document.body).toHaveFocus()

        userEvent.tab()

        expect(checkbox).toHaveFocus()
      })
    ```

  - hover 和 unhover 事件

    模拟 hover 和 unhover 事件

    ```
      test('hover', () => {
        const messageText = 'Hello'
        render(
          <Tooltip messageText={messageText}>
            <TrashIcon aria-label="Delete" />
          </Tooltip>,
        )

        userEvent.hover(screen.getByLabelText(/delete/i))
        expect(screen.getByText(messageText)).toBeInTheDocument()
        userEvent.unhover(screen.getByLabelText(/delete/i))
        expect(screen.queryByText(messageText)).not.toBeInTheDocument()
      })
    ```

- Firing Event
  参考https://testing-library.com/docs/dom-testing-library/api-events#fireeventeventname

## Mock

- 函数的 Mock

  jest.fn()用来 Mock 函数，mockReturnValue 用来设定返回值，无返回值返回 undefined

  ```
    test('测试jest.fn()返回固定值', () => {
      let mockFn = jest.fn().mockReturnValue('default');
      // 断言mockFn执行后返回值为default
      expect(mockFn()).toBe('default');
    })

    test('测试jest.fn()内部实现', () => {
      let mockFn = jest.fn((num1, num2) => {
        return num1 * num2;
      })
      // 断言mockFn执行后返回100
      expect(mockFn(10, 10)).toBe(100);
    })
  ```

  - 具体请参考https://jestjs.io/zh-Hans/docs/mock-functions

- axios mock

  通过 axios-mock-adapter 来模拟 axios 返回值

  ```
    const fakeCompany = {
      testReturn: 1
    };

    mock.onGet('/wsw-hr/api/company').reply(200, {
      message: 'Successfully find Company ',
      data: fakeCompany,
      status: 200,
    });
  ```

  - 具体请参考https://www.npmjs.com/package/axios-mock-adapter

## Redux Test

- 模拟 redux 流程，首先创建 store，然后 dispatch action 去更新 store 的数据。最后做判断，举例

  ```
      test('test redux', () => {

        //创建store
        const store = configureAppStore();

        render(
          <Provider store={store}>
            <MemoryRouter initialEntries={['/my-account/setting']} initialIndex={0}>
              <Routes>
                <Route path="/my-account/setting" element={<AccountSetting />} />
              </Routes>
            </MemoryRouter>
          </Provider>,
        );

        const fakeCompany = null;

        //dispatch action
        store.dispatch({
          payload: fakeCompany,
          type: 'company/updateCompany',
        });

        //查看store
        await waitFor(() => {
          expect(store.getState().accountSettingCompany).toEqual(fakeCompany);
        });
      })
  ```

- 模拟 redux 及 saga

  ```
      test('test redux', () => {

        const store = configureAppStore();

        //模拟axios的put函数
        mock.onPut('/wsw-hr/api/company').reply(200, {
          message: 'Successfully find Company ',
          data: fakeCompany,
          status: 200,
        });

        render(
          <Provider store={store}>
            <MemoryRouter initialEntries={['/my-account/setting']} initialIndex={0}>
              <Routes>
                <Route path="/my-account/setting" element={<AccountSetting />} />
              </Routes>
            </MemoryRouter>
          </Provider>,
        );

        const fakeCompany = null;

        //dispatch new action，此action会在saga中call axios的'/wsw-hr/api/company' API 来更新company
        store.dispatch({
          payload: fakeCompany,
          type: 'company/updateCompany',
        });

        //查看store
        await waitFor(() => {
          expect(store.getState().accountSettingCompany).toEqual(fakeCompany);
        });
      })
  ```

- 测试组件内部的 useState

  组件

  ```
  export const testContent: FC = () => {
    const [count, setCount] = React.useState(3);

    return (
      <>
        <Button
          data-testid="spyOnButton"
          onClick={() => {
              setCount(count + 1);
              }}
        >
          test button
        </Button>
      </>
      );
  };
  ```

  测试文件

  ```
  it('01 test Target Fulfillment Date select options ', async () => {
      const setState = jest.fn();
      const useStateSpy = jest.spyOn(React, 'useState');
      const useStateMock: any = (initState: any) => [initState, setState];
      useStateSpy.mockImplementation(useStateMock);

      render(
        <testContent />,
      );

      const button = screen.getByTestId('spyOnButton');
      userEvent.click(button);

      expect(setState).toHaveBeenCalledWith(4);
    });
  ```
