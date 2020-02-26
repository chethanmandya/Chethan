---
title: Spannable Text
tags: Android
article_header:
  type: cover
  image:
---



####

<img src="https://github.com/chethu/Android-Spannable-Text/blob/master/Spannable_text.gif" width="400" height="800" />




```kotlin
object BindingAdapters {

    @JvmStatic
    @BindingAdapter("setTermsConditionText")
    fun TextView.setTermsConditionText(inputString: String?) {
        val titleText = resources.getText(R.string.termsConditionAgreementMessage) as SpannedString
        val annotations = titleText.getSpans(0, titleText.length, Annotation::class.java)
        val spannableString = SpannableString(titleText)
        val clickableSpanForTermsAndConditions = object : ClickableSpan() {
            override fun onClick(widget: View?) {
                findNavController().navigate(
                    HomeFragmentDirections.showTermsConditionFragment()
                )
            }

            override fun updateDrawState(ds: TextPaint?) {
                ds?.isUnderlineText = false
            }
        }

        val clickableSpanForPrivacyPolicy = object : ClickableSpan() {
            override fun onClick(widget: View?) {
                findNavController().navigate(
                    HomeFragmentDirections.showPrivacyFragment()
                )
            }

            override fun updateDrawState(ds: TextPaint?) {
                ds?.isUnderlineText = false
            }
        }

        annotations.forEach {
            when (it.value) {
                "terms_condition_link" -> {
                    setSpannableText(spannableString, clickableSpanForTermsAndConditions, titleText, it)
                }
                "privacy_link" -> {
                    setSpannableText(spannableString, clickableSpanForPrivacyPolicy, titleText, it)
                }
            }
        }



        this.apply {
            text = spannableString
            movementMethod = LinkMovementMethod.getInstance()
        }
    }

    private fun TextView.setSpannableText(
        spannableString: SpannableString, clickableSpanForTC: ClickableSpan, titleText: SpannedString, it: Annotation?
    ) {
        spannableString.apply {
            setSpan(
                clickableSpanForTC,
                titleText.getSpanStart(it),
                titleText.getSpanEnd(it),
                Spanned.SPAN_EXCLUSIVE_EXCLUSIVE
            )
            setSpan(
                ForegroundColorSpan(ContextCompat.getColor(context, R.color.colorAccent)),
                titleText.getSpanStart(it),
                titleText.getSpanEnd(it),
                0
            )
        }
    }

}
```

```kotlin
      <TextView
				android:layout_width="wrap_content"
				android:layout_height="wrap_content"
				app:setTermsConditionText="@{@string/termsConditionAgreementMessage}"
				app:layout_constraintBottom_toBottomOf="parent"
				app:layout_constraintLeft_toLeftOf="parent"
				app:layout_constraintRight_toRightOf="parent"
				app:layout_constraintTop_toTopOf="parent"/>
```

<!--more-->

