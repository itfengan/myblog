---
title: edittext输入限制
date: 2015-12-13 17:42:12
tags: 
- Android
categories: code
---

# EditText智能限制小数点前后分别保留几位



- 可以分别限制小数点前面几位，和小数点后几位
- 首位输入0时，第二位只能输入小数点
- 首位输入小数点，默认显示0.
- 嘻嘻!

<!--more-->

'''

```java
/**
 *
 * @param frontPoint    小数点前几位数
 * @param behindPoint   小数点后几位数
 * @param editTexts
 */
public static void setEditTextLimit(final int frontPoint, final int behindPoint, final EditText... editTexts) {
    if (editTexts != null && editTexts.length > 0) {
        for (final EditText editText : editTexts) {
            editText.addTextChangedListener(new TextWatcher() {
                @Override
                public void onTextChanged(CharSequence s, int start, int before,
                                          int count) {
                    if (s.length() == frontPoint + 1 && !s.toString().contains(".")) {
                        editText.setText(s.toString().substring(0, frontPoint));
                        editText.setSelection(frontPoint);
                    }
                    if (editText.getText().toString().indexOf(".") >= 0) {
                        if (editText.getText().toString().indexOf(".", editText.getText().toString().indexOf(".") + 1) > 0) {
                            editText.setText(editText.getText().toString().substring(0, editText.getText().toString().length() - 1));
                            editText.setSelection(editText.getText().toString().length());
                        }
                    }
                    if (s.toString().contains(".")) {
                        if (s.length() - 1 - s.toString().indexOf(".") > behindPoint) {
                            s = s.toString().subSequence(0,
                                    s.toString().indexOf(".") + behindPoint+1);
                            editText.setText(s);
                            editText.setSelection(s.length());
                        }
                    }
                    //直接输入一个点,显示0.
                    if (s.toString().trim().substring(0).equals(".")) {
                        s = "0" + s;
                        editText.setText(s);
                        editText.setSelection(2);
                    }
                    //当输入一个0,后面只能输入小数点
                    if (s.toString().startsWith("0")
                            && s.toString().trim().length() > 1) {
                        if (!s.toString().substring(1, 2).equals(".")) {
                            editText.setText(s.subSequence(0, 1));
                            editText.setSelection(1);
                            return;
                        }
                    }
                }

                @Override
                public void beforeTextChanged(CharSequence s, int start, int count,
                                              int after) {

                }

                @Override
                public void afterTextChanged(Editable s) {
                }

            });
        }
    }
}
```

'''

xml别忘了

```xml
android:inputType="numberDecimal"
```



<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=417859220&auto=1&height=66"></iframe>