# brief summary
- objective: provide free mode on teledentistry
    - applied to all patient, not particular patient
- phases:
    1. disable whatsapp notif
    2. fully configurable paid-free mode on teledentistry
        - env var
        - di table khusus -> turn on/off from backyard
- approach for phase 2:
    - (logic) add status **Free** on table order_status
        - (database) use id 7 on table
        - (logic) adjust all payment_status appearance on api, backyard & frontyard
    - (logic) keep using order_id
    - (logic) free mode overwrite envvar TELEDENTISTRY_PRICE to 0
    - (code) adjust pgService->paymentSnapLink to check paid-free mode
        - paid: literally consume Midtrans' API
        - free: doesn't call Midtrans' API
    - (logic) adjust json response for teledentistry/create
        - paid: redirect_url is provided
        - free: redirect_url is null
        
# createAppointment
- validation & set variables
- $paymentLink = teleService->createAppointment(input, files, confirmationStatus, event)
    - validate assignee
    - set payment_option, amount, user_id
    - $paymentLink = pgService->paymentSnapLink
        - create order_id
        - save paymentLog
        - set user_id
        - set payment_option, payment_state, **order_status**, item, amount, expiry
        - get patient
        - recall paymentLink
        - void existing order's status
        - set into $params [order_id, patientInfo, item, qty, amount, expiry]
        - save to DB 
            - $paymentLink = pgService->requestPaymentLink
            - save order, orderHistory, paymentHistory
            - send whatsapp | schedule whatsapp
            - return $paymentLink
        - return $paymentLink
    - set $tele variables (patient_id, date, time, files, order_id, assign_to, video_link, event_id, consultation_id)
    - set [consultation_id, files   ] to $paymentLink
    - return $paymentLink


