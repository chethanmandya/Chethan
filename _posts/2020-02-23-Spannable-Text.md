---
title: Spannable Text
tags: Android
article_header:
  type: cover
  image:
---



####

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

<!--more-->

