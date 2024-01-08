# #仿Medium编辑页

# `仿code`​

参考：[https://juejin.cn/post/7167767301553553438#heading-0](https://juejin.cn/post/7167767301553553438#heading-0)

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Meduim</title>
    <style>
        /* 默认情况下鼠标是箭头，这里要让它变成插入符的样子 */
        [contenteditable] {
            outline: 0px solid transparent;
            cursor: text;
        }

        /* 设置占位符样式 */
        [contentEditable=true]:empty:before {
            content: attr(placeholder);
            opacity: .6;
        }

        .title {
            font-size: 24px;
        }

        .content {}
    </style>
</head>

<body>
    <div class="container" style="width: 100%; height: 100vh;">
        <div class="story" style="padding: 24px; width: 800px; margin: 0 auto;">
            <p class="title" contenteditable="true" placeholder="Title"></p>
            <p class="content" contenteditable="true" placeholder="Tell your story"></p>
        </div>
    </div>
    <script>
        STORY_PARA_FIST_CLASSNAME = 'content';
        window.addEventListener('keydown', (e) => {
            console.log('e', e);
            if (e.key == 'Enter') {
                // 阻止默认自动产生新的div
                e.preventDefault();

                //新建p
                const $newPara = document.createElement('p');
                $newPara.setAttribute('contentEditable', 'true');

                //如果是在标题后按退格，那么第一个p已经变成新的
                //需要把第一个p的placeholder加上，之前p的相关属性去掉
                const $fistPara = document.getElementsByClassName(STORY_PARA_FIST_CLASSNAME)[0];

                $newPara.className = STORY_PARA_FIST_CLASSNAME;
                if ($curr.tagName == 'H3') {
                    $fistPara.classList.remove(STORY_PARA_FIST_CLASSNAME);
                    $fistPara.removeAttribute('placeholder')
                    $newPara.classList.add(STORY_PARA_FIST_CLASSNAME);
                    $newPara.setAttribute('placeholder', STORY_CONTENT_PLACEHOLDER);
                }

                $story.insertBefore($newPara, $curr.nextSibling);
                $newPara.focus();
            } else if (e.key == 'Backspace' && $curr.tagName == 'P') {
                if (!$curr.innerText) { //确认已经退格到头
                    const $previousEle = $curr.previousSibling;

                    // 当前最后一个p
                    if (!$previousEle.tagName) { //最后一个p时返回文本节点，tagName为undefined
                        focusAfterLastChar($storyTitle);
                    }
                    // p大于1个时
                    if ($previousEle.tagName == 'P') {
                        $story.removeChild($curr);
                        focusAfterLastChar($previousEle);
                    }
                }
            }
        });
    </script>
</body>
</html>
```

‍
