# Registration CAPTCHA Flow

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [novel-plus overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)

## 0. Overview

> Stores the generated CAPTCHA text in Redis by client IP for five minutes, then compares it with the submitted CAPTCHA text during registration.

1. Generate CAPTCHA
   - The browser calls `GET /file/getVerify`.
   - `FileController` generates a CAPTCHA image and its generated CAPTCHA text.
   - The image displays that text and is streamed directly to the browser; the image is not stored in Redis.
2. Store generated CAPTCHA text
   - Redis key: `randomValidateCodeKey:<IP>`.
   - Redis value: The generated CAPTCHA text displayed in the image.
   - Set a five-minute expiry.
3. Submit CAPTCHA text
   - The user types the text shown in the image.
   - The browser submits that text as `velCode` through `POST /user/register`.
4. Compare CAPTCHA text
   - `UserController` reads the generated CAPTCHA text from Redis using the client's IP key.
   - Compare the submitted CAPTCHA text in `velCode` with the generated CAPTCHA text from Redis.
     - Different values: Reject the registration.
     - Same value: Register the user and return a JWT.
   - The Redis key is not deleted; it expires after five minutes.
