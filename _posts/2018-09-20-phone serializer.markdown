---
layout: post
title:  "Phone serializer"
date:   2018-09-20 08:43:59
author: Moon Hyungtae
categories: DJango
tags:	django 시리얼라이저
cover:  "/assets/instacode.png"
---

`#django` `#유효성검사`

전화번호 시리얼라이저 모델과 유효성 검사
j.
**ModelSerializer**

기존에 생성되어 있는 User 모델을 참조하여 contact_phone 필드만을 따로 정의합니다. 전화번호 형식에 맞는 값만을 DB 에 저장하기 위해서 정규표현식을 사용.
```python
class ContactPhoneChangeSerializer(serializers.ModelSerializer):
    contact_phone = serializers.CharField(max_length=20, write_only=True)

    class Meta:
        model = User
        fields = ('contact_phone',)

    def validate(self, data):
        new_phone = data.get('contact_phone')
        if not re.match('\d{3}[-]\d{4}[-]\d{4}$', new_phone):
            raise serializers.ValidationError('올바른 전화번호 형식이 아닙니다')
        else:
            return data

    def update(self, instance, validated_data):
        instance.contact_phone = validated_data.get('contact_phone')
        instance.save()
        return instance
```

**APIView 작성**

위에서 정의한 시리얼라이저 모델을 다른 언어를 사용하는 이용자와 주고받기 위해서는 APIView 도 작성하여야만 한다. 검증을 통과(실패) 했을 경우에는 특정 HTTP 상태 코드를 데이터와 함께 보내주도록 한다.

```python
class ContactPhoneChange(APIView):
    permission_classes = (
        permissions.IsAuthenticated,
    )

    def patch(self, request, *args, **kwargs):
        serializers = ContactPhoneChangeSerializer(request.user, data=request.data)

        if serializers.is_valid():
            serializers.save()
            return Response(serializers.validated_data, status=status.HTTP_200_OK)
        return Response(serializers.errors, status=status.HTTP_400_BAD_REQUEST)
```
