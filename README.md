# sslcommerz-payment-api-integration

```php

    function  sslcommerz_event_payment($event_id,$event_booking_id,$user_id,$data=array()){

        $ssl_charge = $this->Settings->ssl_charge;
        $ssl_charge = (float)$ssl_charge;

        $amount = ($data['total_bdt'] * $ssl_charge)/100;
        $amount = $data['total_bdt'] + $amount;

        $post_data = array();
//        $post_data['store_id'] = "test";
//        $post_data['store_passwd'] = "62569E7E1133F59286";

        $post_data['store_id'] = "amir5eff8a9c13365";
        $post_data['store_passwd'] = "amir5eff8a9c13365@ssl";

        // $post_data['total_amount'] = 10;
        //$post_data['total_amount'] = $data['total_bdt'] + $prAmount;
        $post_data['total_amount'] = $amount;
        $post_data['currency'] = "BDT";
        $post_data['tran_id'] = $event_id;
        $post_data['success_url'] = base_url()."event/success";
        $post_data['fail_url'] = base_url()."event/failed";
        $post_data['cancel_url'] = base_url()."event/cancel";
//        $post_data['ipn_url'] = base_url()."payment/ipn/";

        # CUSTOMER INFORMATION
        $post_data['cus_name'] =  $data['name'];
        $post_data['cus_email'] = $data['email'];
        $post_data['cus_add1'] = "Dhaka";
        $post_data['cus_add2'] = "Dhaka";
        $post_data['cus_city'] = "Dhaka";
        $post_data['cus_state'] = "Dhaka";
        $post_data['cus_postcode'] = "1000";
        $post_data['cus_country'] = "Bangladesh";
        $post_data['cus_phone'] = $data['phone'];
        $post_data['cus_fax'] = "";

        # SHIPMENT INFORMATION
        $post_data['ship_name'] = "exsaacscorglive";
        $post_data['ship_add1 '] = "Banani";
        $post_data['ship_add2'] = "Dhaka";
        $post_data['ship_city'] = "Dhaka";
        $post_data['ship_state'] = "Dhaka";
        $post_data['ship_postcode'] = "1000";
        $post_data['ship_country'] = "Bangladesh";
        $post_data['shipping_method'] = "NO";

        # OPTIONAL PARAMETERS
        $post_data['value_a'] = $user_id;
        $post_data['value_b'] = $event_booking_id;
        $post_data['value_c'] = $user_id;
        $post_data['value_d'] = $data['total_bdt'];


        # EMI STATUS
        $post_data['emi_option'] = "1";

        # CART PARAMETERS
        $post_data['cart'] = json_encode(array(
            array("user"=>"Event Registration","amount"=>$data['total_bdt']),
        ));

        $post_data['product_amount'] = $amount;
        $post_data['product_name'] = "demo";
        $post_data['product_category'] = "demo";
        $post_data['product_profile'] = "general";
        $post_data['vat'] = "0";
        $post_data['discount_amount'] = "0";
        $post_data['convenience_fee'] = "0";

//        $direct_api_url = "https://securepay.sslcommerz.com/gwprocess/v4/api.php";
        $direct_api_url = "https://sandbox.sslcommerz.com/gwprocess/v4/api.php";

        $handle = curl_init();
        curl_setopt($handle, CURLOPT_URL, $direct_api_url);
        curl_setopt($handle, CURLOPT_TIMEOUT, 30);
        curl_setopt($handle, CURLOPT_CONNECTTIMEOUT, 30);
        curl_setopt($handle, CURLOPT_POST, 1 );
        curl_setopt($handle, CURLOPT_POSTFIELDS, $post_data);
        curl_setopt($handle, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($handle, CURLOPT_SSL_VERIFYPEER, FALSE); # KEEP IT FALSE IF YOU RUN FROM LOCAL PC


        $content = curl_exec($handle );

        $code = curl_getinfo($handle, CURLINFO_HTTP_CODE);


        if($code == 200 && !( curl_errno($handle))) {
            curl_close( $handle);
            $sslcommerzResponse = $content;

        } else {
            curl_close( $handle);
            echo "FAILED TO CONNECT WITH SSLCOMMERZ API";
            exit;
        }

        # PARSE THE JSON RESPONSE
        $sslcz = json_decode($sslcommerzResponse, true );

        if(isset($sslcz['GatewayPageURL']) && $sslcz['GatewayPageURL']!="") {
            // this is important to show the popup, return or echo to sent json response back

            header('Location: '.$sslcz['GatewayPageURL']);
            exit;
            json_encode(['status' => 'success', 'data' => $sslcz['GatewayPageURL'], 'logo' => $sslcz['storeLogo'] ]);

        } else {
            return  json_encode(['status' => 'fail', 'data' => null, 'message' => "JSON Data parsing error!"]);
        }
    }

    function failed(){
        $this->site->updateData('event_booking', array('payment_status' => 'PENDING'),array('id' => $_POST['value_b']));
        $this->data['data'] = $_POST;
        $this->data['message'] = 'Payment Status Failed. Please Contact With Admin';
        $this->data['page_title'] = 'Payment Failed';
        $bc = array(array('link' => '#', 'page' => 'Payment Failed'));
        $meta = array('page_title' => 'Payment Failed', 'bc' => $bc);
        $this->frontend_construct('exsa/payment/message', $this->data,$meta);
    }

    function cancel(){
        $this->site->updateData('event_booking', array('payment_status' => 'CANCEL'),array('id' => $_POST['value_b']));

        $this->data['data'] = $_POST;
        $this->data['message'] = 'Payment Status Canceled. Please Contact With Admin';
        $this->data['page_title'] = 'Payment Canceled';
        $bc = array(array('link' => '#', 'page' => 'Payment Canceled'));
        $meta = array('page_title' => 'Payment Canceled', 'bc' => $bc);
        $this->frontend_construct('exsa/payment/message', $this->data,$meta);

    }

    function success(){

        $val_id=urlencode($_POST['val_id']);

//        $store_id=urlencode("exsaacscorglive");
//        $store_passwd=urlencode("62569E7E1133F59286");

        $store_id=urlencode("amir5eff8a9c13365");
        $store_passwd=urlencode("amir5eff8a9c13365@ssl");

        $requested_url = ("https://sandbox.sslcommerz.com/validator/api/validationserverAPI.php?val_id=".$val_id."&store_id=".$store_id."&store_passwd=".$store_passwd."&v=1&format=json");

        $handle = curl_init();
        curl_setopt($handle, CURLOPT_URL, $requested_url);
        curl_setopt($handle, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($handle, CURLOPT_SSL_VERIFYHOST, false); # IF YOU RUN FROM LOCAL PC
        curl_setopt($handle, CURLOPT_SSL_VERIFYPEER, false); # IF YOU RUN FROM LOCAL PC

        $result = curl_exec($handle);

        $code = curl_getinfo($handle, CURLINFO_HTTP_CODE);


        if($code == 200 && !( curl_errno($handle)))
        {

            # TO CONVERT AS ARRAY
            # $result = json_decode($result, true);
            # $status = $result['status'];

            # TO CONVERT AS OBJECT
            $response = $result;
            $result = json_decode($result);

            # TRANSACTION INFO
            $status = $result->status;
            $tran_date = $result->tran_date;
            $tran_id = $result->tran_id;
            $val_id = $result->val_id;
            $amount = $result->amount;
            $store_amount = $result->store_amount;
            $bank_tran_id = $result->bank_tran_id;
            $card_type = $result->card_type;

            # EMI INFO
            $emi_instalment = $result->emi_instalment;
            $emi_amount = $result->emi_amount;
            $emi_description = $result->emi_description;
            $emi_issuer = $result->emi_issuer;

            # ISSUER INFO
            $card_no = $result->card_no;
            $card_issuer = $result->card_issuer;
            $card_brand = $result->card_brand;
            $card_issuer_country = $result->card_issuer_country;
            $card_issuer_country_code = $result->card_issuer_country_code;

            # API AUTHENTICATION
            $APIConnect = $result->APIConnect;
            $validated_on = $result->validated_on;
            $gw_version = $result->gw_version;


            $this->db->select_max('id');
            $max_id = $this->db->get('transactions')->row();
            $max_id = $max_id->id+1;

            $ssl_charge = $this->Settings->ssl_charge;
            $ssl_charge = (float)$ssl_charge;
            $charge_amount = ($result->value_d * $ssl_charge)/100;

            // transaction data
            $transaction_data = [
                'member_id' => $result->value_a,
                'amount' => $amount,
                'tr_for_id' => 5,
                'notes' => 'Event Registration Transaction',
                'status' => 1,
                'created_at' => date("Y-m-d H:i:s"),
                'year' => date("Y"),
                'response' => $response,
                'tran_id' => 'EXSAACS'.$tran_id.$max_id,
                'bank_tran_id' => $bank_tran_id,
                'card_brand' => $card_brand,
                'currency' => $result->currency_type,
                'tran_date' => $tran_date,
                'store_amount' => $store_amount,
                'event_booking_id' => $result->value_b,
                'event_id' => $tran_id,
                'charge_amount' => $charge_amount,
                'net_amount' => $result->value_d,
            ];



            $this->site->insert('transactions',$transaction_data);

            $this->site->updateData('event_booking', array('payment_status' => 'PAID'),array('id' => $result->value_b));


            $this->data['data'] = $result;
            $this->data['message'] = 'Payment Status Successfully Paid. Thanks for your payment.';
            $this->data['page_title'] = 'Payment Success';
            $bc = array(array('link' => '#', 'page' => 'Payment Success'));
            $meta = array('page_title' => 'Payment Success', 'bc' => $bc);
            $this->frontend_construct('exsa/payment/message', $this->data,$meta);

        } else {

            echo "Failed to connect with SSLCOMMERZ";
        }
    }

}




```
