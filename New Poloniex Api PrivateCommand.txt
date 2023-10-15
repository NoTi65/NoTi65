#!/usr/bin/perl
# 

# Testscript for Poloniex Private Command - 2023 10 15 -
#use warning;

use strict;
use WWW::Curl;
use JSON::Parse 'parse_json';
use JSON::Parse 'valid_json';
use JSON::Parse 'assert_valid_json';
use JSON::XS 'decode_json';
use Data::Dumper::Names;
use Data::Dumper;
use JSON;
use Date::Parse;
use LWP::UserAgent;
use HTTP::Request;
use HTTP::Response;
use Digest::SHA qw(hmac_sha256_hex); 
use MIME::Base64;



# Variables
my $Command;
my $Sign;
my $temp="";
my $ApiKey="AA_...ZZ";# ApiKey has to be replaced by your ApiKey
my $SecretKey="abc......59z";# SecretKey  has to be replaced by your SecretKey
my $Call;
my $Command1;
my $digest ="";
my $CurrencyPair="ETH_BTC";
$Command1 = 'https://api.poloniex.com/orders?symbol=' . $CurrencyPair  ;

START:



print "Command1 = " . $Command1 . "\n";

$temp = getPrivateCommand ($Command1) ;

print "Result of Sub = "  . "\n";

my $json_out = eval { decode_json($temp) };
print "$json_out" . "\n";
exit;



###############################SUB Routinen ############################################################################################

# Start SUB1

sub getPrivateCommand {
my ($PrivateCommand) = @_;
my $time = time;
my $nonce = qx(date +%s000);# formatting to 5 leading zeros
$nonce =~ s/\R//g;# linefeed remove
my $url ='https://api.poloniex.com';

print "APIKEY " . $ApiKey . "\n";
print "SECRETKEY " . $SecretKey . "\n";
print "Private Command = " . $PrivateCommand . "\n";

$digest=hmac_sha256_hex($PrivateCommand, $SecretKey);
my $encoded = encode_base64($digest);
$encoded =~ s/\R//g;# linefeed remove

$Call =   "curl -X GET  --header 'key: " . $ApiKey . "' --header  'signMethod: HmacSHA256'    --header 'signVersion: 2' --header  'signTimestamp: " . $nonce . "'    --header 'signature: " .  $encoded . "' '" . $url . "'"; 

#different tests - but not working
#$Call =  "curl -X GET  --header 'key: " . $ApiKey . "' --header  'signTimestamp: " . $nonce . "'    --header 'signature: " .  $encoded . "' '" . $url . "'"; 
#$Call =   "curl --location --request POST  '$url'\ --header 'Content-Type: application/json'\ --header 'key:  $ApiKey'\ --header 'signature: $encoded'\ --header 'signTimestamp:  $time'\ "; 


#goto VARIATION1;
#goto VARIATION2;
goto VARIATION3;
#goto VARIATION4;
exit;
#--------------------------------------;
VARIATION1:
print "my Call = " . $Call . "\n";
my  $result = qx($Call); # This will execute the "Call-Command"
print "Result     " . $result . "\n";


VARIATION2:
my $req = HTTP::Request->new(GET => $url);
$req->header('key' => $ApiKey);
$req->header('signatureMethod' => 'HmacSHA256');
$req->header('signatureVersion' => '2');
$req->header('signTimestamp' => $nonce);
$req->header('signature' => $encoded);
print "Request " . $req->content . "\n";

    my $message = $req->decoded_content;
    print "Request reply: " . $message . "\n";

goto FINAL;

VARIATION3:
my $req = HTTP::Request->new(GET => $url);
$req->header('key' => $ApiKey);
$req->header('signatureMethod' => 'HmacSHA256');
$req->header('signatureVersion' => '2');
$req->header('signTimestamp' => $nonce);
$req->header('signature' => $encoded);
my $response = HTTP::Response->new;
my $browser = LWP::UserAgent->new;

my $response = $browser->get($url);
$response->header('key' => $ApiKey);
$response->header('signatureMethod' => 'HmacSHA256');
$response->header('signatureVersion' => '2');
$response->header('signTimestamp' => $nonce);
$response->header('signature' => $encoded);

print "Response " . $response . "\n";
if ($response->is_success) {
    my $message = $response->decoded_content;
    print "Response reply: $message" . "\n";

}
else {
    print "HTTP GET error code: ", $response->code, "\n";
    print "HTTP GET error message: ", $response->message, "\n";
}

goto FINAL;


VARIATION4:
my $ua = new LWP::UserAgent (agent => 'Mozilla/5.0', cookie_jar =>{});

my $request="";
my  $request = $ua ->get($Call)->content;
my %hash = "";
open my $fh, '>', 'text.txt' or die "$!";
printf  $fh "Request " . $request;

for(keys %hash){
	print("Official Language of $_ is $hash{$_}\n");
}



FINAL:

print "\n" . "Result of Sub " . "\n" . $request . "\n";
return $request;
}
# Ende SUB1





