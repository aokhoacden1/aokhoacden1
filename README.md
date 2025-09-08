/** Adminer - Compact database management
* @link https://www.adminer.org/
* @author Jakub Vrana, https://www.vrana.cz/
* @copyright 2007 Jakub Vrana
* @license https://www.apache.org/licenses/LICENSE-2.0 Apache License, Version 2.0
* @license https://www.gnu.org/licenses/gpl-2.0.html GNU General Public License, version 2 (one or other)
* @version 5.2.1
*/namespace
Adminer;const
VERSION="5.2.1";error_reporting(24575);set_error_handler(function($Ac,$Cc){return!!preg_match('~^Undefined (array key|offset|index)~',$Cc);},E_WARNING|E_NOTICE);$Xc=!preg_match('~^(unsafe_raw)?$~',ini_get("filter.default"));if($Xc||ini_get("filter.default_flags")){foreach(array('_GET','_POST','_COOKIE','_SERVER')as$X){$Yi=filter_input_array(constant("INPUT$X"),FILTER_UNSAFE_RAW);if($Yi)$$X=$Yi;}}if(function_exists("mb_internal_encoding"))mb_internal_encoding("8bit");function
connection($g=null){return($g?:Db::$fe);}function
adminer(){return
Adminer::$fe;}function
driver(){return
Driver::$fe;}function
connect(){$Db=adminer()->credentials();$J=Driver::connect($Db[0],$Db[1],$Db[2]);return(is_object($J)?$J:null);}function
idf_unescape($u){if(!preg_match('~^[`\'"[]~',$u))return$u;$ye=substr($u,-1);return
str_replace($ye.$ye,$ye,substr($u,1,-1));}function
q($Q){return
connection()->quote($Q);}function
escape_string($X){return
substr(q($X),1,-1);}function
idx($va,$x,$k=null){return($va&&array_key_exists($x,$va)?$va[$x]:$k);}function
number($X){return
preg_replace('~[^0-9]+~','',$X);}function
number_type(){return'((?<!o)int(?!er)|numeric|real|float|double|decimal|money)';}function
remove_slashes(array$Gg,$Xc=false){if(function_exists("get_magic_quotes_gpc")&&get_magic_quotes_gpc()){while(list($x,$X)=each($Gg)){foreach($X
as$qe=>$W){unset($Gg[$x][$qe]);if(is_array($W)){$Gg[$x][stripslashes($qe)]=$W;$Gg[]=&$Gg[$x][stripslashes($qe)];}else$Gg[$x][stripslashes($qe)]=($Xc?$W:stripslashes($W));}}}}function
bracket_escape($u,$Ca=false){static$Hi=array(':'=>':1',']'=>':2','['=>':3','"'=>':4');return
strtr($u,($Ca?array_flip($Hi):$Hi));}function
min_version($pj,$Ke="",$g=null){$g=connection($g);$Ah=$g->server_info;if($Ke&&preg_match('~([\d.]+)-MariaDB~',$Ah,$B)){$Ah=$B[1];$pj=$Ke;}return$pj&&version_compare($Ah,$pj)>=0;}function
charset(Db$f){return(min_version("5.5.3",0,$f)?"utf8mb4":"utf8");}function
ini_bool($ae){$X=ini_get($ae);return(preg_match('~^(on|true|yes)$~i',$X)||(int)$X);}function
sid(){static$J;if($J===null)$J=(SID&&!($_COOKIE&&ini_bool("session.use_cookies")));return$J;}function
set_password($oj,$N,$V,$F){$_SESSION["pwds"][$oj][$N][$V]=($_COOKIE["adminer_key"]&&is_string($F)?array(encrypt_string($F,$_COOKIE["adminer_key"])):$F);}function
get_password(){$J=get_session("pwds");if(is_array($J))$J=($_COOKIE["adminer_key"]?decrypt_string($J[0],$_COOKIE["adminer_key"]):false);return$J;}function
get_val($H,$m=0,$rb=null){$rb=connection($rb);$I=$rb->query($H);if(!is_object($I))return
false;$K=$I->fetch_row();return($K?$K[$m]:false);}function
get_vals($H,$d=0){$J=array();$I=connection()->query($H);if(is_object($I)){while($K=$I->fetch_row())$J[]=$K[$d];}return$J;}function
get_key_vals($H,$g=null,$Dh=true){$g=connection($g);$J=array();$I=$g->query($H);if(is_object($I)){while($K=$I->fetch_row()){if($Dh)$J[$K[0]]=$K[1];else$J[]=$K[0];}}return$J;}function
get_rows($H,$g=null,$l="<p class='error'>"){$rb=connection($g);$J=array();$I=$rb->query($H);if(is_object($I)){while($K=$I->fetch_assoc())$J[]=$K;}elseif(!$I&&!$g&&$l&&(defined('Adminer\PAGE_HEADER')||$l=="-- "))echo$l.error()."\n";return$J;}function
unique_array($K,array$w){foreach($w
as$v){if(preg_match("~PRIMARY|UNIQUE~",$v["type"])){$J=array();foreach($v["columns"]as$x){if(!isset($K[$x]))continue
2;$J[$x]=$K[$x];}return$J;}}}function
escape_key($x){if(preg_match('(^([\w(]+)('.str_replace("_",".*",preg_quote(idf_escape("_"))).')([ \w)]+)$)',$x,$B))return$B[1].idf_escape(idf_unescape($B[2])).$B[3];return
idf_escape($x);}function
where(array$Z,array$n=array()){$J=array();foreach((array)$Z["where"]as$x=>$X){$x=bracket_escape($x,true);$d=escape_key($x);$m=idx($n,$x,array());$Vc=$m["type"];$J[]=$d.(JUSH=="sql"&&$Vc=="json"?" = CAST(".q($X)." AS JSON)":(JUSH=="sql"&&is_numeric($X)&&preg_match('~\.~',$X)?" LIKE ".q($X):(JUSH=="mssql"&&strpos($Vc,"datetime")===false?" LIKE ".q(preg_replace('~[_%[]~','[\0]',$X)):" = ".unconvert_field($m,q($X)))));if(JUSH=="sql"&&preg_match('~char|text~',$Vc)&&preg_match("~[^ -@]~",$X))$J[]="$d = ".q($X)." COLLATE ".charset(connection())."_bin";}foreach((array)$Z["null"]as$x)$J[]=escape_key($x)." IS NULL";return
implode(" AND ",$J);}function
where_check($X,array$n=array()){parse_str($X,$Va);remove_slashes(array(&$Va));return
where($Va,$n);}function
where_link($s,$d,$Y,$Hf="="){return"&where%5B$s%5D%5Bcol%5D=".urlencode($d)."&where%5B$s%5D%5Bop%5D=".urlencode(($Y!==null?$Hf:"IS NULL"))."&where%5B$s%5D%5Bval%5D=".urlencode($Y);}function
convert_fields(array$e,array$n,array$M=array()){$J="";foreach($e
as$x=>$X){if($M&&!in_array(idf_escape($x),$M))continue;$wa=convert_field($n[$x]);if($wa)$J
.=", $wa AS ".idf_escape($x);}return$J;}function
cookie($C,$Y,$Ee=2592000){header("Set-Cookie: $C=".urlencode($Y).($Ee?"; expires=".gmdate("D, d M Y H:i:s",time()+$Ee)." GMT":"")."; path=".preg_replace('~\?.*~','',$_SERVER["REQUEST_URI"]).(HTTPS?"; secure":"")."; HttpOnly; SameSite=lax",false);}function
get_settings($_b){parse_str($_COOKIE[$_b],$Eh);return$Eh;}function
get_setting($x,$_b="adminer_settings"){$Eh=get_settings($_b);return$Eh[$x];}function
save_settings(array$Eh,$_b="adminer_settings"){$Y=http_build_query($Eh+get_settings($_b));cookie($_b,$Y);$_COOKIE[$_b]=$Y;}function
restart_session(){if(!ini_bool("session.use_cookies")&&(!function_exists('session_status')||session_status()==1))session_start();}function
stop_session($fd=false){$gj=ini_bool("session.use_cookies");if(!$gj||$fd){session_write_close();if($gj&&@ini_set("session.use_cookies",'0')===false)session_start();}}function&get_session($x){return$_SESSION[$x][DRIVER][SERVER][$_GET["username"]];}function
set_session($x,$X){$_SESSION[$x][DRIVER][SERVER][$_GET["username"]]=$X;}function
auth_url($oj,$N,$V,$j=null){$cj=remove_from_uri(implode("|",array_keys(SqlDriver::$ec))."|username|ext|".($j!==null?"db|":"").($oj=='mssql'||$oj=='pgsql'?"":"ns|").session_name());preg_match('~([^?]*)\??(.*)~',$cj,$B);return"$B[1]?".(sid()?SID."&":"").($oj!="server"||$N!=""?urlencode($oj)."=".urlencode($N)."&":"").($_GET["ext"]?"ext=".urlencode($_GET["ext"])."&":"")."username=".urlencode($V).($j!=""?"&db=".urlencode($j):"").($B[2]?"&$B[2]":"");}function
is_ajax(){return($_SERVER["HTTP_X_REQUESTED_WITH"]=="XMLHttpRequest");}function
redirect($A,$Xe=null){if($Xe!==null){restart_session();$_SESSION["messages"][preg_replace('~^[^?]*~','',($A!==null?$A:$_SERVER["REQUEST_URI"]))][]=$Xe;}if($A!==null){if($A=="")$A=".";header("Location: $A");exit;}}function
query_redirect($H,$A,$Xe,$Pg=true,$Hc=true,$Qc=false,$vi=""){if($Hc){$Th=microtime(true);$Qc=!connection()->query($H);$vi=format_time($Th);}$Nh=($H?adminer()->messageQuery($H,$vi,$Qc):"");if($Qc){adminer()->error
.=error().$Nh.script("messagesPrint();")."<br>";return
false;}if($Pg)redirect($A,$Xe.$Nh);return
true;}class
Queries{static$Kg=array();static$Th=0;}function
queries($H){if(!Queries::$Th)Queries::$Th=microtime(true);Queries::$Kg[]=(preg_match('~;$~',$H)?"DELIMITER ;;\n$H;\nDELIMITER ":$H).";";return
connection()->query($H);}function
apply_queries($H,array$T,$Dc='Adminer\table'){foreach($T
as$R){if(!queries("$H ".$Dc($R)))return
false;}return
true;}function
queries_redirect($A,$Xe,$Pg){$Kg=implode("\n",Queries::$Kg);$vi=format_time(Queries::$Th);return
query_redirect($Kg,$A,$Xe,$Pg,false,!$Pg,$vi);}function
format_time($Th){return
sprintf('%.3f s',max(0,microtime(true)-$Th));}function
relative_uri(){return
str_replace(":","%3a",preg_replace('~^[^?]*/([^?]*)~','\1',$_SERVER["REQUEST_URI"]));}function
remove_from_uri($eg=""){return
substr(preg_replace("~(?<=[?&])($eg".(SID?"":"|".session_name()).")=[^&]*&~",'',relative_uri()."&"),0,-1);}function
get_file($x,$Pb=false,$Ub=""){$Wc=$_FILES[$x];if(!$Wc)return
null;foreach($Wc
as$x=>$X)$Wc[$x]=(array)$X;$J='';foreach($Wc["error"]as$x=>$l){if($l)return$l;$C=$Wc["name"][$x];$Ci=$Wc["tmp_name"][$x];$wb=file_get_contents($Pb&&preg_match('~\.gz$~',$C)?"compress.zlib://$Ci":$Ci);if($Pb){$Th=substr($wb,0,3);if(function_exists("iconv")&&preg_match("~^\xFE\xFF|^\xFF\xFE~",$Th))$wb=iconv("utf-16","utf-8",$wb);elseif($Th=="\xEF\xBB\xBF")$wb=substr($wb,3);}$J
.=$wb;if($Ub)$J
.=(preg_match("($Ub\\s*\$)",$wb)?"":$Ub)."\n\n";}return$J;}function
upload_error($l){$Se=($l==UPLOAD_ERR_INI_SIZE?ini_get("upload_max_filesize"):0);return($l?'Unable to upload a file.'.($Se?" ".sprintf('Maximum allowed file size is %sB.',$Se):""):'File does not exist.');}function
repeat_pattern($og,$y){return
str_repeat("$og{0,65535}",$y/65535)."$og{0,".($y%65535)."}";}function
is_utf8($X){return(preg_match('~~u',$X)&&!preg_match('~[\0-\x8\xB\xC\xE-\x1F]~',$X));}function
format_number($X){return
strtr(number_format($X,0,".",','),preg_split('~~u','0123456789',-1,PREG_SPLIT_NO_EMPTY));}function
friendly_url($X){return
preg_replace('~\W~i','-',$X);}function
table_status1($R,$Rc=false){$J=table_status($R,$Rc);return($J?reset($J):array("Name"=>$R));}function
column_foreign_keys($R){$J=array();foreach(adminer()->foreignKeys($R)as$p){foreach($p["source"]as$X)$J[$X][]=$p;}return$J;}function
fields_from_edit(){$J=array();foreach((array)$_POST["field_keys"]as$x=>$X){if($X!=""){$X=bracket_escape($X);$_POST["function"][$X]=$_POST["field_funs"][$x];$_POST["fields"][$X]=$_POST["field_vals"][$x];}}foreach((array)$_POST["fields"]as$x=>$X){$C=bracket_escape($x,true);$J[$C]=array("field"=>$C,"privileges"=>array("insert"=>1,"update"=>1,"where"=>1,"order"=>1),"null"=>1,"auto_increment"=>($x==driver()->primary),);}return$J;}function
dump_headers($Nd,$hf=false){$J=adminer()->dumpHeaders($Nd,$hf);$ag=$_POST["output"];if($ag!="text")header("Content-Disposition: attachment; filename=".adminer()->dumpFilename($Nd).".$J".($ag!="file"&&preg_match('~^[0-9a-z]+$~',$ag)?".$ag":""));session_write_close();if(!ob_get_level())ob_start(null,4096);ob_flush();flush();return$J;}function
dump_csv(array$K){foreach($K
as$x=>$X){if(preg_match('~["\n,;\t]|^0|\.\d*0$~',$X)||$X==="")$K[$x]='"'.str_replace('"','""',$X).'"';}echo
implode(($_POST["format"]=="csv"?",":($_POST["format"]=="tsv"?"\t":";")),$K)."\r\n";}function
apply_sql_function($r,$d){return($r?($r=="unixepoch"?"DATETIME($d, '$r')":($r=="count distinct"?"COUNT(DISTINCT ":strtoupper("$r("))."$d)"):$d);}function
get_temp_dir(){$J=ini_get("upload_tmp_dir");if(!$J){if(function_exists('sys_get_temp_dir'))$J=sys_get_temp_dir();else{$o=@tempnam("","");if(!$o)return'';$J=dirname($o);unlink($o);}}return$J;}function
file_open_lock($o){if(is_link($o))return;$q=@fopen($o,"c+");if(!$q)return;chmod($o,0660);if(!flock($q,LOCK_EX)){fclose($q);return;}return$q;}function
file_write_unlock($q,$Jb){rewind($q);fwrite($q,$Jb);ftruncate($q,strlen($Jb));file_unlock($q);}function
file_unlock($q){flock($q,LOCK_UN);fclose($q);}function
first(array$va){return
reset($va);}function
password_file($h){$o=get_temp_dir()."/adminer.key";if(!$h&&!file_exists($o))return'';$q=file_open_lock($o);if(!$q)return'';$J=stream_get_contents($q);if(!$J){$J=rand_string();file_write_unlock($q,$J);}else
file_unlock($q);return$J;}function
rand_string(){return
md5(uniqid(strval(mt_rand()),true));}function
select_value($X,$_,array$m,$ui){if(is_array($X)){$J="";foreach($X
as$qe=>$W)$J
.="<tr>".($X!=array_values($X)?"<th>".h($qe):"")."<td>".select_value($W,$_,$m,$ui);return"<table>$J</table>";}if(!$_)$_=adminer()->selectLink($X,$m);if($_===null){if(is_mail($X))$_="mailto:$X";if(is_url($X))$_=$X;}$J=adminer()->editVal($X,$m);if($J!==null){if(!is_utf8($J))$J="\0";elseif($ui!=""&&is_shortable($m))$J=shorten_utf8($J,max(0,+$ui));else$J=h($J);}return
adminer()->selectVal($J,$_,$m,$X);}function
is_mail($rc){$xa='[-a-z0-9!#$%&\'*+/=?^_`{|}~]';$dc='[a-z0-9]([-a-z0-9]{0,61}[a-z0-9])';$og="$xa+(\\.$xa+)*@($dc?\\.)+$dc";return
is_string($rc)&&preg_match("(^$og(,\\s*$og)*\$)i",$rc);}function
is_url($Q){$dc='[a-z0-9]([-a-z0-9]{0,61}[a-z0-9])';return
preg_match("~^(https?)://($dc?\\.)+$dc(:\\d+)?(/.*)?(\\?.*)?(#.*)?\$~i",$Q);}function
is_shortable(array$m){return
preg_match('~char|text|json|lob|geometry|point|linestring|polygon|string|bytea~',$m["type"]);}function
count_rows($R,array$Z,$ke,array$td){$H=" FROM ".table($R).($Z?" WHERE ".implode(" AND ",$Z):"");return($ke&&(JUSH=="sql"||count($td)==1)?"SELECT COUNT(DISTINCT ".implode(", ",$td).")$H":"SELECT COUNT(*)".($ke?" FROM (SELECT 1$H GROUP BY ".implode(", ",$td).") x":$H));}function
slow_query($H){$j=adminer()->database();$wi=adminer()->queryTimeout();$Ih=driver()->slowQuery($H,$wi);$g=null;if(!$Ih&&support("kill")){$g=connect();if($g&&($j==""||$g->select_db($j))){$te=get_val(connection_id(),0,$g);echo
script("const timeout = setTimeout(() => { ajax('".js_escape(ME)."script=kill', function () {}, 'kill=$te&token=".get_token()."'); }, 1000 * $wi);");}}ob_flush();flush();$J=@get_key_vals(($Ih?:$H),$g,false);if($g){echo
script("clearTimeout(timeout);");ob_flush();flush();}return$J;}function
get_token(){$Ng=rand(1,1e6);return($Ng^$_SESSION["token"]).":$Ng";}function
verify_token(){list($Di,$Ng)=explode(":",$_POST["token"]);return($Ng^$_SESSION["token"])==$Di;}function
lzw_decompress($Ia){$Zb=256;$Ja=8;$eb=array();$ah=0;$bh=0;for($s=0;$s<strlen($Ia);$s++){$ah=($ah<<8)+ord($Ia[$s]);$bh+=8;if($bh>=$Ja){$bh-=$Ja;$eb[]=$ah>>$bh;$ah&=(1<<$bh)-1;$Zb++;if($Zb>>$Ja)$Ja++;}}$Yb=range("\0","\xFF");$J="";$yj="";foreach($eb
as$s=>$db){$qc=$Yb[$db];if(!isset($qc))$qc=$yj.$yj[0];$J
.=$qc;if($s)$Yb[]=$yj.$qc[0];$yj=$qc;}return$J;}function
script($Kh,$Gi="\n"){return"<script".nonce().">$Kh</script>$Gi";}function
script_src($dj,$Rb=false){return"<script src='".h($dj)."'".nonce().($Rb?" defer":"")."></script>\n";}function
nonce(){return' nonce="'.get_nonce().'"';}function
input_hidden($C,$Y=""){return"<input type='hidden' name='".h($C)."' value='".h($Y)."'>\n";}function
input_token(){return
input_hidden("token",get_token());}function
target_blank(){return' target="_blank" rel="noreferrer noopener"';}function
h($Q){return
str_replace("\0","&#0;",htmlspecialchars($Q,ENT_QUOTES,'utf-8'));}function
nl_br($Q){return
str_replace("\n","<br>",$Q);}function
checkbox($C,$Y,$Ya,$ve="",$Gf="",$cb="",$xe=""){$J="<input type='checkbox' name='$C' value='".h($Y)."'".($Ya?" checked":"").($xe?" aria-labelledby='$xe'":"").">".($Gf?script("qsl('input').onclick = function () { $Gf };",""):"");return($ve!=""||$cb?"<label".($cb?" class='$cb'":"").">$J".h($ve)."</label>":$J);}function
optionlist($Lf,$th=null,$hj=false){$J="";foreach($Lf
as$qe=>$W){$Mf=array($qe=>$W);if(is_array($W)){$J
.='<optgroup label="'.h($qe).'">';$Mf=$W;}foreach($Mf
as$x=>$X)$J
.='<option'.($hj||is_string($x)?' value="'.h($x).'"':'').($th!==null&&($hj||is_string($x)?(string)$x:$X)===$th?' selected':'').'>'.h($X);if(is_array($W))$J
.='</optgroup>';}return$J;}function
html_select($C,array$Lf,$Y="",$Ff="",$xe=""){static$ve=0;$we="";if(!$xe&&substr($Lf[""],0,1)=="("){$ve++;$xe="label-$ve";$we="<option value='' id='$xe'>".h($Lf[""]);unset($Lf[""]);}return"<select name='".h($C)."'".($xe?" aria-labelledby='$xe'":"").">".$we.optionlist($Lf,$Y)."</select>".($Ff?script("qsl('select').onchange = function () { $Ff };",""):"");}function
html_radios($C,array$Lf,$Y="",$xh=""){$J="";foreach($Lf
as$x=>$X)$J
.="<label><input type='radio' name='".h($C)."' value='".h($x)."'".($x==$Y?" checked":"").">".h($X)."</label>$xh";return$J;}function
confirm($Xe="",$uh="qsl('input')"){return
script("$uh.onclick = () => confirm('".($Xe?js_escape($Xe):'Are you sure?')."');","");}function
print_fieldset($t,$Ce,$sj=false){echo"<fieldset><legend>","<a href='#fieldset-$t'>$Ce</a>",script("qsl('a').onclick = partial(toggle, 'fieldset-$t');",""),"</legend>","<div id='fieldset-$t'".($sj?"":" class='hidden'").">\n";}function
bold($La,$cb=""){return($La?" class='active $cb'":($cb?" class='$cb'":""));}function
js_escape($Q){return
addcslashes($Q,"\r\n'\\/");}function
pagination($E,$Gb){return" ".($E==$Gb?$E+1:'<a href="'.h(remove_from_uri("page").($E?"&page=$E".($_GET["next"]?"&next=".urlencode($_GET["next"]):""):"")).'">'.($E+1)."</a>");}function
hidden_fields(array$Gg,array$Qd=array(),$_g=''){$J=false;foreach($Gg
as$x=>$X){if(!in_array($x,$Qd)){if(is_array($X))hidden_fields($X,array(),$x);else{$J=true;echo
input_hidden(($_g?$_g."[$x]":$x),$X);}}}return$J;}function
hidden_fields_get(){echo(sid()?input_hidden(session_name(),session_id()):''),(SERVER!==null?input_hidden(DRIVER,SERVER):""),input_hidden("username",$_GET["username"]);}function
enum_input($U,$ya,array$m,$Y,$uc=null){preg_match_all("~'((?:[^']|'')*)'~",$m["length"],$Ne);$J=($uc!==null?"<label><input type='$U'$ya value='$uc'".((is_array($Y)?in_array($uc,$Y):$Y===$uc)?" checked":"")."><i>".'empty'."</i></label>":"");foreach($Ne[1]as$s=>$X){$X=stripcslashes(str_replace("''","'",$X));$Ya=(is_array($Y)?in_array($X,$Y):$Y===$X);$J
.=" <label><input type='$U'$ya value='".h($X)."'".($Ya?' checked':'').'>'.h(adminer()->editVal($X,$m)).'</label>';}return$J;}function
input(array$m,$Y,$r,$Ba=false){$C=h(bracket_escape($m["field"]));echo"<td class='function'>";if(is_array($Y)&&!$r){$Y=json_encode($Y,128|64|256);$r="json";}$Zg=(JUSH=="mssql"&&$m["auto_increment"]);if($Zg&&!$_POST["save"])$r=null;$od=(isset($_GET["select"])||$Zg?array("orig"=>'original'):array())+adminer()->editFunctions($m);$ac=stripos($m["default"],"GENERATED ALWAYS AS ")===0?" disabled=''":"";$ya=" name='fields[$C]'$ac".($Ba?" autofocus":"");$_c=driver()->enumLength($m);if($_c){$m["type"]="enum";$m["length"]=$_c;}echo
driver()->unconvertFunction($m)." ";$R=$_GET["edit"]?:$_GET["select"];if($m["type"]=="enum")echo
h($od[""])."<td>".adminer()->editInput($R,$m,$ya,$Y);else{$Ad=(in_array($r,$od)||isset($od[$r]));echo(count($od)>1?"<select name='function[$C]'$ac>".optionlist($od,$r===null||$Ad?$r:"")."</select>".on_help("event.target.value.replace(/^SQL\$/, '')",1).script("qsl('select').onchange = functionChange;",""):h(reset($od))).'<td>';$ce=adminer()->editInput($R,$m,$ya,$Y);if($ce!="")echo$ce;elseif(preg_match('~bool~',$m["type"]))echo"<input type='hidden'$ya value='0'>"."<input type='checkbox'".(preg_match('~^(1|t|true|y|yes|on)$~i',$Y)?" checked='checked'":"")."$ya value='1'>";elseif($m["type"]=="set"){preg_match_all("~'((?:[^']|'')*)'~",$m["length"],$Ne);foreach($Ne[1]as$s=>$X){$X=stripcslashes(str_replace("''","'",$X));$Ya=in_array($X,explode(",",$Y),true);echo" <label><input type='checkbox' name='fields[$C][$s]' value='".h($X)."'".($Ya?' checked':'').">".h(adminer()->editVal($X,$m)).'</label>';}}elseif(preg_match('~blob|bytea|raw|file~',$m["type"])&&ini_bool("file_uploads"))echo"<input type='file' name='fields-$C'>";elseif($r=="json"||preg_match('~^jsonb?$~',$m["type"]))echo"<textarea$ya cols='50' rows='12' class='jush-js'>".h($Y).'</textarea>';elseif(($si=preg_match('~text|lob|memo~i',$m["type"]))||preg_match("~\n~",$Y)){if($si&&JUSH!="sqlite")$ya
.=" cols='50' rows='12'";else{$L=min(12,substr_count($Y,"\n")+1);$ya
.=" cols='30' rows='$L'";}echo"<textarea$ya>".h($Y).'</textarea>';}else{$Si=driver()->types();$Ue=(!preg_match('~int~',$m["type"])&&preg_match('~^(\d+)(,(\d+))?$~',$m["length"],$B)?((preg_match("~binary~",$m["type"])?2:1)*$B[1]+($B[3]?1:0)+($B[2]&&!$m["unsigned"]?1:0)):($Si[$m["type"]]?$Si[$m["type"]]+($m["unsigned"]?0:1):0));if(JUSH=='sql'&&min_version(5.6)&&preg_match('~time~',$m["type"]))$Ue+=7;echo"<input".((!$Ad||$r==="")&&preg_match('~(?<!o)int(?!er)~',$m["type"])&&!preg_match('~\[\]~',$m["full_type"])?" type='number'":"")." value='".h($Y)."'".($Ue?" data-maxlength='$Ue'":"").(preg_match('~char|binary~',$m["type"])&&$Ue>20?" size='".($Ue>99?60:40)."'":"")."$ya>";}echo
adminer()->editHint($R,$m,$Y);$Yc=0;foreach($od
as$x=>$X){if($x===""||!$X)break;$Yc++;}if($Yc&&count($od)>1)echo
script("qsl('td').oninput = partial(skipOriginal, $Yc);");}}function
process_input(array$m){if(stripos($m["default"],"GENERATED ALWAYS AS ")===0)return;$u=bracket_escape($m["field"]);$r=idx($_POST["function"],$u);$Y=$_POST["fields"][$u];if($m["type"]=="enum"||driver()->enumLength($m)){if($Y==-1)return
false;if($Y=="")return"NULL";}if($m["auto_increment"]&&$Y=="")return
null;if($r=="orig")return(preg_match('~^CURRENT_TIMESTAMP~i',$m["on_update"])?idf_escape($m["field"]):false);if($r=="NULL")return"NULL";if($m["type"]=="set")$Y=implode(",",(array)$Y);if($r=="json"){$r="";$Y=json_decode($Y,true);if(!is_array($Y))return
false;return$Y;}if(preg_match('~blob|bytea|raw|file~',$m["type"])&&ini_bool("file_uploads")){$Wc=get_file("fields-$u");if(!is_string($Wc))return
false;return
driver()->quoteBinary($Wc);}return
adminer()->processInput($m,$Y,$r);}function
search_tables(){$_GET["where"][0]["val"]=$_POST["query"];$wh="<ul>\n";foreach(table_status('',true)as$R=>$S){$C=adminer()->tableName($S);if(isset($S["Engine"])&&$C!=""&&(!$_POST["tables"]||in_array($R,$_POST["tables"]))){$I=connection()->query("SELECT".limit("1 FROM ".table($R)," WHERE ".implode(" AND ",adminer()->selectSearchProcess(fields($R),array())),1));if(!$I||$I->fetch_row()){$Cg="<a href='".h(ME."select=".urlencode($R)."&where[0][op]=".urlencode($_GET["where"][0]["op"])."&where[0][val]=".urlencode($_GET["where"][0]["val"]))."'>$C</a>";echo"$wh<li>".($I?$Cg:"<p class='error'>$Cg: ".error())."\n";$wh="";}}}echo($wh?"<p class='message'>".'No tables.':"</ul>")."\n";}function
on_help($kb,$Gh=0){return
script("mixin(qsl('select, input'), {onmouseover: function (event) { helpMouseover.call(this, event, $kb, $Gh) }, onmouseout: helpMouseout});","");}function
edit_form($R,array$n,$K,$bj,$l=''){$fi=adminer()->tableName(table_status1($R,true));page_header(($bj?'Edit':'Insert'),$l,array("select"=>array($R,$fi)),$fi);adminer()->editRowPrint($R,$n,$K,$bj);if($K===false){echo"<p class='error'>".'No rows.'."\n";return;}echo"<form action='' method='post' enctype='multipart/form-data' id='form'>\n";if(!$n)echo"<p class='error'>".'You have no privileges to update this table.'."\n";else{echo"<table class='layout'>".script("qsl('table').onkeydown = editingKeydown;");$Ba=!$_POST;foreach($n
as$C=>$m){echo"<tr><th>".adminer()->fieldName($m);$k=idx($_GET["set"],bracket_escape($C));if($k===null){$k=$m["default"];if($m["type"]=="bit"&&preg_match("~^b'([01]*)'\$~",$k,$Wg))$k=$Wg[1];if(JUSH=="sql"&&preg_match('~binary~',$m["type"]))$k=bin2hex($k);}$Y=($K!==null?($K[$C]!=""&&JUSH=="sql"&&preg_match("~enum|set~",$m["type"])&&is_array($K[$C])?implode(",",$K[$C]):(is_bool($K[$C])?+$K[$C]:$K[$C])):(!$bj&&$m["auto_increment"]?"":(isset($_GET["select"])?false:$k)));if(!$_POST["save"]&&is_string($Y))$Y=adminer()->editVal($Y,$m);$r=($_POST["save"]?idx($_POST["function"],$C,""):($bj&&preg_match('~^CURRENT_TIMESTAMP~i',$m["on_update"])?"now":($Y===false?null:($Y!==null?'':'NULL'))));if(!$_POST&&!$bj&&$Y==$m["default"]&&preg_match('~^[\w.]+\(~',$Y))$r="SQL";if(preg_match("~time~",$m["type"])&&preg_match('~^CURRENT_TIMESTAMP~i',$Y)){$Y="";$r="now";}if($m["type"]=="uuid"&&$Y=="uuid()"){$Y="";$r="uuid";}if($Ba!==false)$Ba=($m["auto_increment"]||$r=="now"||$r=="uuid"?null:true);input($m,$Y,$r,$Ba);if($Ba)$Ba=false;echo"\n";}if(!support("table")&&!fields($R))echo"<tr>"."<th><input name='field_keys[]'>".script("qsl('input').oninput = fieldChange;")."<td class='function'>".html_select("field_funs[]",adminer()->editFunctions(array("null"=>isset($_GET["select"]))))."<td><input name='field_vals[]'>"."\n";echo"</table>\n";}echo"<p>\n";if($n){echo"<input type='submit' value='".'Save'."'>\n";if(!isset($_GET["select"]))echo"<input type='submit' name='insert' value='".($bj?'Save and continue edit':'Save and insert next')."' title='Ctrl+Shift+Enter'>\n",($bj?script("qsl('input').onclick = function () { return !ajaxForm(this.form, '".'Saving'."…', this); };"):"");}echo($bj?"<input type='submit' name='delete' value='".'Delete'."'>".confirm()."\n":"");if(isset($_GET["select"]))hidden_fields(array("check"=>(array)$_POST["check"],"clone"=>$_POST["clone"],"all"=>$_POST["all"]));echo
input_hidden("referer",(isset($_POST["referer"])?$_POST["referer"]:$_SERVER["HTTP_REFERER"])),input_hidden("save",1),input_token(),"</form>\n";}function
shorten_utf8($Q,$y=80,$Zh=""){if(!preg_match("(^(".repeat_pattern("[\t\r\n -\x{10FFFF}]",$y).")($)?)u",$Q,$B))preg_match("(^(".repeat_pattern("[\t\r\n -~]",$y).")($)?)",$Q,$B);return
h($B[1]).$Zh.(isset($B[2])?"":"<i>…</i>");}function
icon($Md,$C,$Ld,$yi){return"<button type='submit' name='$C' title='".h($yi)."' class='icon icon-$Md'><span>$Ld</span></button>";}if(isset($_GET["file"])){if(substr(VERSION,-4)!='-dev'){if($_SERVER["HTTP_IF_MODIFIED_SINCE"]){header("HTTP/1.1 304 Not Modified");exit;}header("Expires: ".gmdate("D, d M Y H:i:s",time()+365*24*60*60)." GMT");header("Last-Modified: ".gmdate("D, d M Y H:i:s")." GMT");header("Cache-Control: immutable");}@ini_set("zlib.output_compression",'1');if($_GET["file"]=="default.css"){header("Content-Type: text/css; charset=utf-8");echo
lzw_decompress("h:M±h´ħ̐±ܔ͈\"PѩҭcC³鈞d<̦󡼤:;NBqR;1Lf³9Ȟu7%©d\\;3͇AФ`%Eá¨¬e9&㈰r4Mض2\r&:©Φsꆓ0웪3Máº䭻LC@茩:dt3-8aI\$ã°꥝§	ˣ9lT!Ѻ>e\0ʤĉdõC±玺6\\c£A¾Àrhڍ4뫷ԢΚ|O+f9ɆX屷h\"얓i¶¨ú¼|ǫ9衅ƣ©̐-4W~T:¹zkHᠢ{ Э&єtª:ܸ.K v8#\",7!pp2¡\0\\Á¼ \$β7 ތ#𐎩\"��̔ (L#:\0Τ˸؝½̘Fǉd&Ϊv ¨°ڗ£@düE¯ۺÿÀ!,9.+`JᐡhbDP<Á°|\"¨򢨃p쾿ˑ+b2	L¡黳FώQ´|¦©ºrÁKlɏԟƴ=ɏ𢙀K|©͐®ºª\r=Ӓ¬> 谱£(ö¼¯k袉JU,PUumI.t聭KüXý4²z)MPЧkºٳe`¾N>D#¹\\(YTۣ@÷hLű]ȴͺNKն2\\73i/V¤¯lۖŅYÁҗBA/[J֝ĘВ\r;'²2^텪Ԣ¹ۣ3锽0Hö8\r�¹kf׃ϝq՗)똆²Cÿ2`A°¾82Á!À¾hmǟвGD£ú¼2-C öYc`¼¾s ȶù2µ¶9u渎y̒Mgy�CZO~^3¥Ҝ2<¡Ы0£wM®{d#`Zۀú㞺±빂涯C%»¼ª=Rqذ¼_+쵭ۋ>��'GÀ򁡜$�¡^j><ögf¾hømb*/\$\$lµÀشµg)Aj׳ w򞣡£ÁԕõTN՝Tÿ¾㥚ÿùjJ¦ªCf4ƲẆৈ* xª¹ª¬֌ho&k\r䟅,­᲎Ժ>s(úLAs땀®¸5Ct¥©n6¤㋠ll\\9D圜\!mv\0ʏA{9`.¸⠗¡Slg6¡ۈ!.2Á0Pؔ ѩ\r\$7w ¿⻇°Ɯ$0❃IÞJ\n猷Pc¼'*Eh°䢉;µpނÛ(x:DȌˮj±9AC@ I3jf5sI`X}ƜҔ#ºශT`d¤jhH49S\nqHJQ ؈`,FPʰ\\{¨m\r̾@20u!	\$PoQ¥4ǚΜnZL捂°Bꂁk)@¶d³铕LȰv χy²ЃBߌ^o˃䛪¾R\"ҽӢ#ºrͥS;\r4驪ùI©攉°rહ=6¡ȑT\0\0įf#¤ù=\$՞¢҈6P晃:®G\$ ݰ蹺a3Hz;GÜr!hJ±nз	ýoYÿú¨¨¥􁗌v̛i|ùĥ-❂¤d\$pD䑒2T\rհaU§øn5rªjÁ\$r%D©ǩ\$GڃBuΠ:²ø`¥(l ӄ)I	Àցع窪\rtв¡ ƺI§g[Xûc,u\rvJ5?§ª҃\":^.uڊùPo\$t\$18ҜnnKĔ%EZ,6턈V��¹i«&z¶«xpdrx*}ʐR25+ѓf첇w»qþ0X1ϲdXߢ茏W©會V8f\"냐q(uퟔ� GqM#а񣋕3WA޶Y􌧉ÞeK]t¾]E«j=SX@ςӇ\rӘ\$喔9¤ܬ0ؐ7\"D.匎<;ªNj潵ºüÀ垨mmڒG¤68 ƝC%v'ªy߆k/^5쁑3䆀ͮڛka*Dºߑ³:ѷÿC}ýĠø`�ݷε筼	3Рiթ¨½¯4\0.:®Qߌ礘ͨ»܄f⧙%䝩M	¥¡نY3\0Ǚ##tP6(BϤ§ ©ȯy¯6­|ȵþ¸IH7 ⶦz?ù(њŖ\$«RWT¼谦:û(ºᠲ϶i s=D\\,kr1􆙓2ؠ걃A乘¢&nÁ~͌Ҭ£6;«vp ӞM#ȝ«¿´Ù؎AũJ.þܡ»t焮Ys횏Pwĸ¦m⦚ݿÀAU㊷Jퟻ�z»3\$P��U9,©֊󳒣©µ5Pnb¦㵎ѐ{N`ꩥ۝°iw\rbº&Eµ\\tg÷𢾡1+mȰw#vlǽUԶ¡Ҝ0弄.(wb@þ´\\°w()´腟¼¢;䚩]/&˃|>Q¶\"c	<F\rѕ7܉叵\\'ʑSfe÷\rR🅖lo/.\n֠F챯蠷ehýԥ񃪗 ȞTٳa4³2ಆ ֠oΜ\¿A?ö]퟼�B[±{7' ݑ%¨6ýs7\$Ó򉋾%»u)յi ÷ÿ0¡#¥£\rƦ䏐ˍXN\\ɇ\0­¤ﬔü�To\0{oĂRr^§䮃 @Z.C,þ£û'°J-ꂇL \rP«CD\"ꢌ ^.\"ڳʆÀh¤\0؈ވ؜r\0n`	   n o	Ԝr ´\r ¢0À` 0£¢	Á\rp 	0À\n F@`ϠV\0 \n¢\r\0¤\njÀ̜n@ \0ܜr \nÀ	 ޜn@ʋ@ܲ\0𦠫\n@̏ @ƈ ງ­Ƃ*©wб0g£5𡇐xG		\n¥\n­µ½\rpŌ͜rpӜrÁ	П0朲뎰󏠢\r@퟼�º 쉠°鞜r ퟙ� d@𳠭ѱQ9сB¨<®tȱN?ѓá°v-aퟌ�𓉰ۗP§\n°¯°¿0ǌ°Ϙ°ՙ\0@љЭõ©Ѱ\r ^À䏜"i@\nÀ 6 \0ꉠp\n򜮀` qޒQD�䑍°d9񔕱÷1û Ö2	򏜮2²rR#°Ј2+\r/գ¡@񜢆 ֋Q\rÀ܈\řÀȆ@\n h\n〪퀆\0̈`¨	ÀƆ@±!񻓱Co敒2õ򛟱e Qk ±p ߡP±3с!Ør%ҁۙpÀ	 ͬа`쌰 `\n@ꌠff ° ª`Ǝ \nঋ@´	↣Ȉ`p�䐿o¿ \\%Bl»ϔ?瞍-jP񳕲¤3/ӳ*QlpÀ	p\r`=Μn\0_>±1'񑣜0߾\0¡\rԀÀ䈠 ¢\n@⎀ fʰÁ'±@ĀބÀʅ\0脜rdFhµI\$󠘄謜򤸩烍ȕPݔT>ʘ7\0]E̣ʉDG¼©́C´\\BMDԈ򥍦md訜r􏇠F熩DNn2鴴ΔwFt톊�وCԈu+°菜$K¬6蔓E䐤.AKԏL*1JK>º©莈񍔁H¨ø\"GNPjęE´>젫H&5HԷLM#E͐c¶c8欏 £¢,ÿ¢µC⦎PtǀV tü\nþԖݴõI	kG΁H	¤)D(JPl1jn𬔍¬܊�*&𮜜ٕHUfLk¯K��Fº쇼|HNx\\ Nlꎤ򴉢֜0rzMtU|Zꚴĸ¦õº唘蜲HCþ΀B\"怳b¦cnA Ɗ9Ort´A4¸\rªـh򀁞`۞¥V0՞!L򪕐þ𜛮µ^\r\"±¬쩚kaõ쬰d ־©}\0NԾIGPն˕ûYtyMPr䔋YԒEڔ¥xĔĩ6`נjg´ޏµ1SB²܂°蕲하ָV?Id[Iµߑ.喂펝`񩬮̲vԞɂU)ԜΠӜn2PV~㌠¢¦\"\"&§µr]-¢ ᰪ¨\0f\"Kj`ϱ\nJ㜢q¥F¬.öú\"@r«¨³`򎤳q>\"��\r\$ح£ ¢R1̜ªh&Hú`Z¬V	u+Moʬ¬\n3JꜲ Đא2Iü ©D'א!S0W0J?yûpjZ.·\n\r ÷pw\"-+㺗r!`żv2\nl¢f(¤m<̽↜r©Q}Ͼ7濅̜r·ࣥ½oγ÷ｷ옌x<ø~׽W¬øiEãۂ8\n bjj돿\r: ¶)v֪'{·Ֆ翱\no{·±)C߂iÀꜲø%·遀ˠʃ(ثk􋸉��d¾ ¿øjXLN÷(A}xeø|ø±wø´ÁGxh䭄Xx¦\rԥK¾ö޼oqx¸»8s4e¸xρ¤펮Ӫ4Føc8~hIp]⻅厥ù( ·Ӓø𼥖÷ࣸC¹Bػwؽ¹Oƣøù}¹Q8[ױ[{cT%ù&´ʯ·Á¹:*b酔`陙mIYWk8Yo§ùu¸)¹Y5o9㞦÷ى¹<¦8(ù?\0[s׀*8··}¹ߟ9g\rӟ¹¹✰\n'w±x)݌٩µº92·Z1ù[Iº+¢÷_57=D§qz!}ºK¤ùNd壚3\0拣qº+C¡غ¿Y_g8؋úy¸½ډ¨Kⴚ{ٓ¨82ZzX\0Ϩz©úS§ؿªڱ¹e« ¾\rª>¾:£§ù­ڇ¬´_¬ZÏ£e»¬úµ¨®:稸¿u÷­{ȕMڡ°ؖ킫zɈû㢲YSWJ(wOwӁwm تZN÷l¶勧C̦9姙�Ц8BDʌ¤6©£Zy±x{膦;!©[m¯¬ۻ}»¸)¯¸#δ¶[®´Ũ½b½ ɘ¸«úՆu¨û­««¹ʬO¥\"Fª7y?»9£¼ٮdѽ»±¹{ݍs½{¹ e´ʦ>\"֣c§¬d¤ޒcs{þ̃vdCN½[Àû¹GM¿C瓭ɄE@");}elseif($_GET["file"]=="dark.css"){header("Content-Type: text/css; charset=utf-8");echo
lzw_decompress("h:M±h´ħƃȨ0ÁLЁह1¢S!¤ۉF!°䶽0̦󡼥	G2Ύa'3IЊdK%Ӊҭ(\r&㘨Я;NBɄ깾2Sª*¶^#ƆQб=¥Ɗ¶W^L£¡˯¡̆c!Ħ³­6»m⾡¯³l4&1Lf³9Ȟu7VDc3خ82IΆ°ʬ:5ʘr·P«ö1 ĭ¡>5W/Fc©Dh2L\rN¯˝Wo&Ĩkޥ2ٌЀb12Ƽ~0РㄽN¶\0úf4MC魗쮂=´p㏩Zش²NӾͻѭC 򦥪z¢99P醁㤜"­²߻\0f񆪸์¡pԺm܋8ㄢ@\nXຎrس#¨ػ­CÀ[¹Cx#𐉲\\\"¡ÀpƑÀ]#¨Ӕ5R¬r6#Ì7𰟡H\$\$IRdۧˉ8]	§xܩ+򔦾Ń@-㨛;îܢ／䩲Ô𝎴,«㔣-Mr¥6IcX4¹aʛõKETh@1@팍R®K9\r£(湌#8ˇ¡CpwID5̲ج\"_'ӊUB̕¡9c@Ç=C\n蜛SȰ§ՠj®ם7PUȚ£۹J]®<׋\nƲσzû?B÷Բܴ͒\r/P\rÁM[X¡F_쿪¬HӢnC&¡f%@cC^.2㸨׎Cѽ^sw½L퟉�/ø5OنM业³	*XbͮIgʔ&󡱄݊>皆N彭`湬䴥s»៓j\\&:Sa吻��²H뻎®XΞ¯餏¡kt?.´õ±,ZOÁ·@@8Z3©c\"Ѩß\nؽAH1\\ZϞ/k꿅΃L�C\\񣞩0OüÍͯlpr7䜲qÁ¶ٗRaƛ¡¥ضc@Áwmk/۸£*?ǅ̐贪5朜m§¡k๾d1n𫕑#£§ܸ¾w禆«Lo&hĪPrnR,5��\"\$3»dYH(p\rALAC)pTõPl²!\"L´8Àദ\0µ±0P8׆ûܣɊ	`¨e0	®ژ1û	®DĊs°H³)kƠ¡[ųԃȹpjx,\rAm!ٙ<h1䜢);}elseif($_GET["file"]=="functions.js"){header("Content-Type: text/javascript; charset=utf-8");echo
lzw_decompress("':̢Фi1㳱ԝ	4̀£̑6a&󐰇:OAI줥:NFᄼݡCyꭲ˅\"㉔ʲ<̱ي/C#ٶ:DbqSeJ˦Cܺ\n\n¡ǱS\rZH\$RAܞS+XKvtdܧ:£�EvXŞ³jɭҩejײM§©亁B«ǈ&ʮL§C°3呰ՌƩ-x蜮ә섓ȂyNa䐮:煛¼䨳͐( cLŜ/õ£(ƈ5{޴Qy4øg-ý¢ꩴڃfЎ(իbUýϫ·܎&㺎ä��b¾¢ؠ.­ہ\rΐܼ»σĺ¼͜n ©ChҼ\r)`蘥`淥CʒȢZùµ㘊<QűX÷¼@·0dp9EQüf¾°ӆ؜r䡍拨h��Ünp'#Č¤£H̔(i*r¸榼#¢淋Ȉ~# ȓA:N6㰊©lլ§\r􁊐γ£!@Ҳ>Cr¾¡¬h°N᝞¦(a0M3Ͳ׶ԕ愣E2'!<·£3R<𛂏㘒攃Hη#n䫱a\$!蜲Ј0¤.°wd¡r:Yö¨酲慡]<¹j⥳@ߜ\װl§_\rÁZ¸ғ¬TͩZɳ򎳏\"²~9À©³jㅉPؖ)QYbݕD동c¿`zᣞµѨ̛'룴BOh¢*2ÿ<ŒOꦧ-Z£գ 踎aОú+r2bø\\ᾰ©ᾌ¥ùש¸Áޮٞp!#`卫Zö¸6¶12׃@鲫yȆ9\r줂3烰ޅ輣!p9஑o6s¿𣆘3홠bA¨ʶ񹦽ÀZ£#6ûʥ?s¨Ȝ"ω|؂§)þbJc\r»½Nޟsɛih8ϐ¹束躞;躈垌õuI5û@豍ªA萗aH^\$H׶ㅖ@ÛL~¨ùb9'§ø¿±S?PЭ¯򘰍C𜮘R򎭌4ޓȓ:Àõ܇Ը򄌴µh(k\njIȶ\"EY#¹Wrª\rG8£@tСXԓ⌂S\nc0ɫC I\rʰ<u`A!󩎐Բփ¢\0=¾ 斡䐈1ӢK!¹!埐pĝIsѬ6⤞éɎi1+°ȏ┫꼕¸^	ᜮɲ0´Fԉ_\$멦\0 ¤C8E^¬į3W!א)u*䟑Ԩ&\$ꔲY\n©]Ek񄚖¨\$xTse!RY» R`=L򸣄ޫ\nl_.!²V!r\nHЫ²\$א`{1	|± °i<jRrPTG|w©4b´\r¡ǆ4d¤,§E¡ȶ©䏼è[Nq@Oi׾'ѩ\r¥󗻦]#潐0»ASIJdс/QÁ´⸵t\r¥UGğG<鍼y-Iɺ򄤝М" P B\0ý텀ȞÁq`Aa̡J堒䊮)JB.¦TܱL¡÷ Cpp\0(7cYYa¨M鱕em4Ӗc¢¸r£«S)o񍠂p权!I¼¾Sb0m챎(dEHø¸߳Xª£/¬P©踙yƘ鸵Ȓ\$+֖»²gd茀öΎyݐ܏³Jט렢lE¢ur̬dCX}e¬셑¥õ«m]в ̽Ȩ-z¦Z庻Iö\) ,\n¤>򩷞¤朲VS\njx*w`ⴷSFi̓d¯¼,»ᐖZFM}Њ À\\Z¾P읠¹zؚûE]�ɟO룭ԁ]À ¬Á%þ\"w4¥\n\$øɺV¢SQDۺݶ«䇋wMԮS0B-sƪ)㾚�ǁ2δA;慥n©Wz/AÚh G~cc%˛Ʉ£&lFR昷7|ªI¢3¹�֌a½䣃0RJ2ϑ%³ÆẠSà©L½^ trڮٴ񃋛¡ʩ;Ǯ喚Ł>ùá[®aN»¤Ϟè!g@1𰳢üN·zԼb靖䛑õO,۳Cuº¸D״j޹I;)®݀霮n䣺ሂ툗<sµ	Ŝ0÷hN¼PӹΘ{ue¤ut뵕ö腰º󧽋 3򇮽g¥뺏¸΂ӊ썃º򗑇0øط9p-Àº	ý§ø˰ٿ'5»´\nO۷e)Mȩ_kҙz\0V´֚ú޻j\nÀ¦꧸Րf-䠃ˮ@&]#\0ڶp𹍖ƛtˤú¶ ó¼b}	G1·m߲u߀*񟀸D²3Ǳ¼BӳQ淵ús%Ꜯª5s§ut½»s򹥀øN¯4¥,J{4@®þ\0»PĊÞº=¯l²`襏~F١h3o酜"¤q·R<iUT°[Q഑UǍ6üT. º갧pe\\¼½��ߖ̕pCe	ٕԗ\"* M	¨¦Dþ±?ûhüز¡УzU@7°CӴýaµ²iE!f˜$üB¤<9o*\$¯掬H\$ ŀࠊ怐\rNÀYn<\$²	ÀQ=F&¥ *@]\0ʌϋ W'd֠z\$愐jА[¢ö\$򤯐0#& _̠+B)wv%	┄LcJRSÀ©`̌Ů	FW	ꋜnBP\n爜r\0}	瑩0²Z𸌂򯠪\$«: §8ieüÀ؏xↈ¹± ¬GnøsgO¢䕥VU°@NÀ¤ϐúd+®(oJXƨຍ'F٣רV®I^٢1>݀Ќ\"¤ ȑ񒡑\\¢`[¥¤«¨.ذfbF;낇瘆pϰ/t` 􋮨§ÀV鸸 bȲ(HlÁΔ¯1v­ޅퟡ�1T񓙱ɚ1¦Ѫf˜nT\$°頎q+˜�vևüVmûǲ°¨اϸ±񧌥«\"Lm¼((CLz\"h☘m= \\H\n0U f&M\$¤g\$񕅠a\rPþ>`ˣgªh��R4Hѧ穏­³²GK;\"M¶ۨThµBEn\"b> ڜrÀ©#\0敎:�	QQ1{	f:B˂ᒪ&ܣ)JµĂr¹+.\$Аqõ-r®S%TIT&Qö·һ#2o(*P¯ⵯ`1H®¢'	<T𤱷ª¾sÀ쬎ڊ Ҏɔ샞\r%3r&Ǔ4B쯜0ЫLH\$³4dӾү³ඵHö·* ºù3Jǐ¥<Hh©pú'珯&ﲉ.V.¢s5ӥ3�Zۨõ9Eg§;R;±J½QÀªӶgz@¶޳'dZ&¬U㲟򦆠戢*²D򈡠䜲;%x'G#°͠wÁ#°֠ȏ2;#򂶀Xɢa힜nb{4KG¦ߥ°҇uE`\\\rB\r\0¨-mW\rM\"¶#E��Fծzӳÿ@4JȒ[\$ʫ%2V%􏦔ԖdմhemN¯-;Eľ%E¥E´r <\"@»FԐ·L ܟ­ܴEɰ°҄z`е7雎4¯˜0°F:hΌKh/:\"MʖZԶ\r+P4\r?¤SøO;B©0\$FCEpǍ\"%H4D´|LNFtEѾgþ°5彊\r\"޼5³õ4ྋ񐜲bZਜr\"pEQ'DwKõW0g'l\"hQFùCc®򉈒P hF]5µ& fT挩STUS¨ÿ4[uºNe\$oüK윏 ࿢\" 5DũEҥ\"±]®/­∐J­6U¤ÿ`õ񡩿V-0DӔbMͩ­¯ؽċ`楱ELt+웶C7j뤵¤:´V4ơ3ߒ\rG򉔮#¥<4-CgCP{V\$'놈ӷg໒@䧐²S=%À½󆱫: ¢kع®²¤󚎥]aO¼҇9;6ۜ⸗À¨*øx\"U®YlBö򯎰ֆ´°·	§ý\nퟦ�ɬҚm\0񵢲䮰Oq̨̍bʗ1s@йK麭pE¦Spw\nGWoQӱG}vpw}q񱓜\ƷƒZ÷@̬¡týtƻpG}w׀/%\"LE\0tÀh⩧\rʏڜ\W@ɧ|D#S³¸ƃVϢR±z2Ϗõövµú©	㽨¢¯(¸\0y<¤X\rםx±°q·<µIsk1S񭎑4Yq8.ֹS;qˡ,'(򛃖伮豈J7H眢.³·¨񵌏°ü#ʄQ\rerÀXv[¬h\$⻭阙 °ûJBg韉iM8¸'nƘtDZ~/b֕8¸\$¸¸DbROOƻ`O5S>¸öΛ DǪ¸¥䇀_3Xø)©À'鄊d\rX»©¸UD애X8򕸯-旅Ѕ̎` 	জnZ˔@Ra48§̺ø©\0銸°֎§\\갥㌅·f\\ 𾜢@^\0Zxڟ\0ZaBr#嘇𜲕¨{˕¹flFb\0[ވ\0[6	¢° ©=✮ ¦WBøƜ$'©kG´(\$y̥9Ҩ8٦ h®Rܔ٦o؈¼ ǇøY£4ش7_­dù㹍'ý¢ú ܺﲻz\rٖ  尾vG菸莘썏h'樎XöS0³\0\0ʉ¸ý9s?öI¹MY¢8ؠ9𘼤£HO,4	xsP¤*G¢磘8·ªQɠøwB|Àz	@¦	ݔ£9cɋ¤¤QGĢFjÀXúoSª\$dFHĂPÀѧ<嶴Ŭ}ﮭ£rÿ\"ŧk`¡cࡸ¦e»C¨у쎬:¼ޘ:X̠¹Tނ´dƃqh¤γÆ¹ׂLvʒ®0\r,4µ\r_vԞL򏪃¥jMᢛ  ퟼�sÀޏZ°@øº䁶;f�2Yc륺'MerʛF\$ȡꪜn ¤	*0\rºAN»LP¥䪙»»¿¼;ƣVӑ|(ퟧ�Ċ[p8󺼼Ԟ\r䂦/DƔؕҞ Bퟘ�_¶N5M�� \$¼\naZЦ¶ȋ~ÀUleõrŧrҚZ®aZ³¹ㇸգs8RÀGZ w®¢ªN_Ʊ«Yϣ򭭉⪀]¦;ƚLڿº¶cøû°ŰƚIÀQ3¹O㇇|y*`  굉ڴퟳ�v8#¯R��X͢V𶸆«i3Fõׅдدc82ۍ­\"¶¹©G¦Wb\rOЃ¿Vd蓭¤w\\䍯*cSiÀQү㳒`úd7}	º)¢ϴ·,+bd§۹½FN£3¾¹L\\㾥Rn\$&\\r��敝O5kq,&\"DCU6j§p燉\\'@oµ~赎=¨|&贡ϕBطHڹyz7ϐ·(Ǎø⇽b5(3փ_\0`zТ®Уr½8	𢚠vȸL˓·)²Sݍ<²*7\$º\rRb·₥ýƴDszϒ>[Q½Ќ&Q«¨À¯¡̧\rpp̺·/<}L¢#°ΕТZ¹㲜"tƯ\n.4ާ櫐ºp®D쮠¥ʹNȢFआ\0`^夎\rnȂ׳#_℆ w(ü2÷<7-ªX޹\0··s¬ø,^¹hC,塺מ\rKӮ䝓¢¯Ţ﹔؜\򫶂Z઎\0§Q9eʛ˞Eöw?>°\$}£·D#ª𣠣ӄ0MV3½%Y»ۀ\rûĴj5ԅ7¼ü{ŝLz=­<븉øMõ°õ⇘ю޿LŜ$Ე{(ÿpe?uݬR*Xº4鮽¿͜0\"@}<.@õ	ގ²²\$Ujsݯü\"* 補$Ծ÷զCPI	ÿ贿᎞ùü¦蠆´	퟼�\\ ̟莑5YH@٢④Ѩù櫱O0T©' 8¡wü»­öj+Hv_#º�06ȷ֎X໤+£ܓ\\À喜n\0	\\🟂>sӏA	PFöd8m'@\nH´\0¬c菷SߟؑY᠝²¨¢RלýDna\" 왾¿Ám퟼�@6䔒½+잇xV䜰°WӰnw.¡؃b«9Í¸EȼE·Â\rЈr¬\"иx¸-¸ꊃ⚜rN6n·\$Ҭý-B툦^ө┥y&㣗Wǧ࿢vR쉸¥³N\0°Àn≔`8X¬ퟅ�:{Oþ@\" !Á¤\$K䓱oЋjY֪J´þ­ܨ}d<1IǌxdʙΔT4NeeC0䥿:DFڵLު::HjZ嗓­FõRªMրnS\n>PO󛌜$V8;#K\\'ùB֨»R®د°Rџ8Áj骄Ej \\~vƂжěp@TX\0002dE	H퇖𱗄\"Q'EDJB~A´A¤Il*'\n¶Y宨+©9¾񒙄pgү\"¸18İIAʆCȨV*a萀d֐£5H\" A؛嶃ݳ¬Yݘ;訞ȯ¨¸0㖶}y\r͒⎗¥1u\"ˋmざ񟌺0焄`ߏ¯¿\\B1^\nk\r]lhø}]HBW`±0½꨹rFf)W,Ւ§]sm9'O¢xԽͬ깊8§£? 4ɉ¡\"҅蛽̼ѭS¨ɃþMûж̱6y|Z򁋨%ࡑ#8¢TC!pºˇ\nدCZ(ｷ阡 ·Á?9|󝰼BL\r\n]ÀPB0¤&+tĈ񖅘Ăx^÷L𽛦Ă񸽽ЏruЋ\0¾\0005哀\"Uؔ@܆°\0\$䁁ސ\"ҠČ]l/	ù퉢B4¯.¶ d7\r@=ª߬¢՛*G j¬üf`»:Hn쑔bĀ71Ǫ)C<@A͙#°¦¡둥o▒Y!ŊIDM¼\nlt¨/)\\43)®ٲ­ɸөÁ²f[ ppp1µ©#Ð¶p\0ą§œlÀ^{A¤TH儶ցʓ«蜮\0P∀.\r|ÀTFD0Sy𛀏'1ց´¤K² dؓµ±¯ĎBþC禅)þWs Hee+@4 r·ᛚ*Lp1<üfNY'­-	XKVa¦L­¥ö\"\"쬕£qɮYJHୗ HV/lCަȀH)oÁ&\\2ă­%⡩z\n^Q(6옄 ȻJq°᫜00a#˶\0vr,»M̺&Aԅ򬃜»9%YdBꃨÀֆ!W\0ꢜr{ƀǱ¹I¬22AڒکH¾a@r0Gɜ7Dd.LM<ひЈˬk/Meª¹}Ғ3作0ЦɏBø\nPd.\"ȱF3Xȓd(*¨J6 䇙F:¬ש1±ῬQ&Ϙùµ¬h<J͋¤fdEպ*񏁸\n\0¼À.\"B -#£ÀΗt¿IΫõЉI8 ²8dh	«胸§~°	L!K(úBXµ£-ȬhΥc/ֲ旋PՉõ«Nʲȼɧ׶Ҽ\"µM'¡K,\\H°ɥ5*o]4҆P	2ͼ)T¾oÀ\n¢¸؉¶ڢġ¨(ø_8Xr绅uú؎Jù¡鏐[rûDC:¸@ÁͳÀl\0©e\\*x@Aȡ&�5×,ª#1xÀ º!T Dª­(Qᄊ|D D:\0Ɂِ¹ԠÁbaEӿrn°²WkxøX=i,\$3[r9BƱ§d㡘þ\0ºԖH4­«ɼ(zʆº��IbJ©g Un(}¨J\"ϦABб9~ŉ霣ڜ$¹%d  e\"µ`À존ª¨'O= À @\$µO\nmTׯ+Z䱙ø-­¢ꟐF?ҕ_I¤J˘ ģ2¢ꭖ¶;ª?2¥Áᰡ*P3Ɏªõ럔<E¥JŜ\(ݲ��ةꉑ鬩·󉒌L&¥͡ȯKÁiцt»¤°΄KúHRl¢ȬEs¶¿¤DøxǴ¬i¾º֡faBɱ󼆔˥>V皎©ɭQjŷ§þ\"%RhȠg£ፌ³ø«խb£58R¨¯Ī㝧9Ԇꊰ«·Թ¤2Q0ý¬IR[üZ£ݎ\0÷ǂ20£¡\\[@ᑜ0¤Ԋxùµ䅃{©✤lp1=\0·RоɾE~߆ꗙѕ:0À%R+)\0°	ƑQဨ\"¡_jTX\0윲1\0P9#\0ʹ򈻂ª|À²LöZ¼Ƃ6ù/BܮB{񰁠|HĔ,ቪ;(õ`ʲ@6ª>¡	忄P\0/¹󜰼\\ťBܠjq©U/\rc©üꁐԒ¤6(N\0º/\$ܮ8µj*U\$񺊹*³=¬;🜤f¬➸X؂CEþr\"/ͪkځ%\\9k§ù肚𰧆­À(¬𜧴U��®m¤@kT\0Վ¹EᕍsEhy򥜮䩠)b7ª㐨W%,ȈJ¤r¨󲄶rhEù\n0Q곊 U9TPOÀմ°8j|¤}Ò<0ȋ♃Zl Еؔᶗ°șڇ*¯\$οÀU\rۜ"¤.ª Ts~˾(𳁔aº¨@իlࠃ:Π­:OiùBXÁ?ʄ¦鷉¾Lj|ͺnK:ز}²\0݉UMc`P%nn\n,촡Q'%+H.苃\"#Gг`¥¡ݨݜn1fg\0М'¼k¦²qxD<\",a|{~þ󸜃<S»iBnkNþ և³}Ӡk:ή­Àݧ۩JD°hÛf¢\"kV~³ᗭM`HOkD¬^0/tj«l³\r!Ϧ<ÀG��ºնµ#@­ek@2«w齴0ܜ­tęį1ĵ̹vː%8±?1¼ۊl旁xtǔmp­fK3Z܊£=\0@^p·¹¶泸]Ҳ'봙¡@C·b뎔圲[ȣV��À˯-¦ݠe·}À附ª܉-釭m³I\0+͖߄ۛB+稐-ٴ䫾®q蝖i>=½0-¦cLpJ b\ndᲩ⫣ᇋs­·䞜"ґЎø`.úȍԹȐEtPqԉ]󤫊8¼»rWTŁIµ苑f÷aG.떄7y狿lٕA³7'¥1	Ⓚ-ٸI§m·˂L:eΉAƗøݎ¶EIݢWzԳW򽂟°)*/)Cʇÿx*c]쥷}½∅»_όIvͲ½'\$U÷ݓ4k5WʏJC® 7*b%<WC@ƉÀ¼©»c{޴«򔋑¬3)X򘦦¢eL쟉坢,N젲k#p5 ´f4«öǺ돺¯#⽤\\®ºࡻNøbԕ𯹛𖀈Sմ¾`qӿ~1=언厉¸*ᏏJꖃ¡񮌢ڨ'Dd,@kL񚹠¤÷\\〉2ͩĪ±<³@_q÷2\0ձÁ)`À꽕s°±󍆙\0¡ӢÀֈ\n­F׼*Àx*À렐ԠÁ-\rø|@ѱԷ𖈚@w󝿈H]µ嘜0¶༟w¾µh0!˳¢1Ϗ¾¦ǬhW°.È꽗ªR*÷A_Ƃ兄Է ?1,Ub̹=tȴè¤䍗¢^夙;ߎ豌@򨱼D⅌Hx©T()0z`џл¨AL錱)\n̋[fHWo@bBKÀiM±ä+ﾨvI¶(z:䝮݀À 9uiѤDY֢¾ûɏ`ö®ᝉ\0°RĆ,K,÷¨㶌¸Ĝ"\"£1gª(­|T.,񹇶b+\rk]u¶&詼©墣Sͅd[¼,gꨡJº(CĶk¤\rF؂+	񌐹₌©¹))UA߂Uh§ң3x񭮹±úü»䘋xȘ®2¯´q¬ibֲY7逫̹즈, §¼Ω¬٪¤J:«N¸ܒcly\nõ¼2ŗ��>Ŷ6Q#A0­ꆻέiù~@VXÀ¢^¿山-ɫ϶|£ܝVf¸¢û.{	ᒀ\r·§;걬p°/لõuFǑd\$PЮ0=@kSƔ0hÁɈ@ы/*(O斮´G>(rˎ!6ષ®򙽘Z@º²'&06kE|ϧ|H;¼掲gҥ˗+¦¯4ù;̓¯¯'x|f©9­̚(O¨𤦧鷷%9]¦צ}̃G֔ĳ¦µ炾󓌅÷XM0ͩgQ·ª¶8̄ù+O}¶͝0}9֐ޖ»ߎhίmgD铳°ü¦Ġ\nͷ4勳P~}O)©Ug܏9ùɃ֪§8PݸÁ(Хĳö۪޷oABאi)üK򄽙u¤봠}s±1软dݖ[Ĵ\n¬粺lMзr:F#{ְ*#°xܜ°¯<Ds½k/mw :^櫦≱¿ďD¨2ºz*Ѳnª%��ک⃙ *ʡ8-·ᦴH'텄㜲͐ºд䝄8`¿\"¡»»i]ZZ>Z\0ަ9û욑+䟂~ᜤޭLĐ\\쇁XA©¬ 耳̓i秺Ҩ$÷SMڔ'1ׅ菄͢	˵E©\0Ğ\$㴴Ԯ¥쇺\rMƷS¦ӖlsªAf֋૬Nlۄ^zz²dS®/rt²Nù>ýo%i¥½\0J¯B©po¢ܒê/֘٫x\ny+«쬥4α5Q'JD]¿B@mӎ´ȃR§Ski~ú܎¶t0盠1z	&׻^\nOՒ¶²ɖ÷뒀³GV@T*ވ9яG0\0'֠Ѱ\r廢QKsLd*;\nၗ恂.ĔUNpଁL—Rԥ øbFÀøyn> IKÀ¶rGû	@ٌ?cIݓu%GöO��փöh¦5TüyüI­ٺ\\0¼إː>öʊ0˞¾ûQB¶©EI/-LBTڡb6쿫`jp\0K¾kd⍄/䉓k.+*ͻ¡R|gR¡ýøW\\wùtன¤^Zc8՚~FÀ°SǵԃSm̕;b>\0jz='Á>̥qy}:»u§µ&垀WºDQ¢ϣ-ª˾Ƕ<[e÷xؠ萮[ú¹L©\0wmùl°tz맼S&ퟬ�dbܸͺoi⧋©\r`ւµԿD5u@bN¸ϕ𤷤�ԛõ裀񻃎r鉞ût±¾󜰯ŁtMscBW?°*D.põ¤'2Ge\rp*#­e¹ЪڅCýӸ\"³QI\nhiøQÁ@ᜲl	߈´߮¤ʴ*វøsÁ9ퟂ�hqժ¸~,¤ᙎ¸ĤQs¦\rBjºõDÿǡ 񼼔)C´\n¶ø°ͦ¹D{\rЕlְѭR㋊\r@rk§閏¢ø +Z�P¾ۖΨ鵸Ȩ􇟀ڳ㙈ø󏯧#䊧ȓu\$F&\n-v\"P܎涛jnnt뱐ߖ®§»¥öꁷbx߄Ąѵ፭԰³a\0\r§/!ȉ¢Ѻ�ިᮄGf-Mdna^(e¤¨·YޏZ,SEöN\\§ճ¸=Ҵ~Mʹ¸\rƫý҆tŦ񵜢|`ђEᲀR󃺜̠⻄耓k/K晚¹®3sJ¡䃿5XGͪ%®9)QϠ£Qܤᦱth¶��R沱ш¢ڑݜrC吏E԰#w燗2/¾֯炿齞 /Ժ񜎐΂ę˅¬\0{+򼀴+¨䱟бª斉ʹ·|ú÷ȕv갱ª¹Ԉƌ&Ϝr\\떟 =°EbګnOβnꑘ({ɹuzK­¯`=:ø\núğ÷\0ªꇐ[饙:pq+¦ԒldY띜"Ň[Vϵ{H-­ÁHןý⢸j떆Ք5ܢ\0\"N?E;+°O~»wNÝ;L'턓OFԪ䁻±D歗!#sNɼժ��Ѿmu³¤ȳG¯8ûΔn]¶¼Ρ:úzIMn O°8À脝z5o\\57<ōŲ#8⨱鿳Nیõ¸	}úx&4盠z½��¶ɽ¡<*W¸蕝󀥽{HZ§±,(<oԯÀxW¨t¶2퐝# A*·¡»o\\缟R²}xH>NP¸|Qɚ|x°'ȭ° ۅ2\0 ݿƾ2*\r|]töpᅜ"¢ڲJuuXybބ\nʚ|H7 _򗑮þGuXyH>T\r¨G»侅Ql¼񨉟§n!ε'ĪºC5¸ݾUª2!b	£9Pw4弛õᢽy藞|񢎋a\$¾g骁 󔇕ː¡&~9(\\*½!b_Ϲûw±7\\£ǰ]=ߜ\*䞭@𣎷ͪ证5QN`@<\0°6!9Ƒl¥\$wI\$4 õ¾2뜤¥&Ь.RZ򠗳Yuyᤳ찥&SI®݀¨EJiLcõºV®1F񱅤Z\r\r¦̨¡kڂ»öHH񋿮ªªöKý§ ?xµ⭰\n۪d͎3K󖄃ӵ9)ľ:B#¨̤N5A1ƉƸ̏d[3ڠᨑ[s~)±9 DN⹸᱾>‘±'ȽΏH貧,½\"¥󰻜0˱eo>¦۽®|«2¦G+B¨@z·Ϥø򄀝}rQÁҠk/|픇񺗑¯䗜0硂4>򕞼õg݃oûXE买üŌrgAĶ¼p¿eú1ﴪŇ룔½7ڀ[ö>]ý#뿪B¨~փ/¿}ųÿ:ûU\$𿼼¿Gü䡿\n>0#!i>.{A}'hQÿLw뾟W_¨h#dÀŃ»ªdFQ¸µ󢪻渜"\"¤P{õགྷ޴ Nוө󟭕՜r_Ŋ؄e?l4À2¡?\n嗆ú	崱Ε½°_ݿ`_üõǠjý¬{_k_ۿo÷~ÿ¿c*#ÿ(´/ü!Dn¤Fÿ`＿@s􂚡®?;܅Ⲻÿþ¾ÿ\0kþ	ÿ*Ný삄;¼õ°+d\nZZdB»À÷ `B5搜n8¬֩క̐c#ou½¤k߁ˊMݯw.쁪FÀJ¦ȡ|®Ĉ2FcY).¬§º􁘈y򁛫꾈ù#/&¢£ö㛀 ÿ񘂌Y@ý¨À(|\r\0,O¼񰙢Ԏ²Ŭ0דۡˑÀɌ A\$Ǻ0,ˀªӰ>>9úÁ\\tiø<\0㗁q\0Ľ@`񜰦Vj°­dߠ'(	!_²nõ 0+c´µiig8a]'=-¬B!(§ظ_ݫƸ²j©µ)\rH5Hn	,f«r�d\$򖈐 ¬2n鴆ܛȽ୭d©FE-dᩨaN_z4@À[讣\$x!!i0TªʵÀ8̉¸¼¯þ\0PZ8ZÁ¹ꃣ砕Á®+ЊAAF(䇁؛`mg*¸vS, ǆܞퟘ�Aþ۬ &Ğ¨9ꀅÁücݰw+nν°)\$뎅Б𾁊ۡ朰004\0u񻄨´¤\$°­y	!°B۠A<µaAz ¨ÁZA4\$ZY9.aX\rdځL¶|oOz|߂Zܚ(횣ĆÀ");}elseif($_GET["file"]=="jush.js"){header("Content-Type: text/javascript; charset=utf-8");echo
lzw_decompress("v0F£©̐==ΆS	Њ_6MƳ訲:ECI´ʯ:CXc\r昄J(:=E¦a28¡x𸿔ħi°SANNù𸳅NB጖l0瓉˕l(D|҄犐¦À>E㩶yHch䂭3Eb堸b½߰EÁpÿ9.̾\n?Kb±iw|ȠǷd.¼x8EN¦㡔͒23©ᜲљ̨y6GFmY8o7\n\r³0²<d4E'¸\n#\r򈱸讅C!Ğt訓õ͢qH¢sÿ2Nq٤̔9÷À#{c랥µÁ옳nӸ2»Ár¼:<+̹CȨ®Ün<��`ȶ/b蜜 ȡHزSڙF#8Јǉ78Ë«*ں!À詎旫¨¾:+¯ù&2|¢:㜢9ʁږ:­Ў§¶㰁/#À 0D᜜±'Ǳؓ漢2a@¶¬+J⼮£c,ø£°1¡@^.Bܑ᠏K=`Bΐ趏 ξ(eK%! ^!ϬBȡHSs8^9ͳ¤O1ѮXj+⸮M	#+օF£:7Sڜ$0¾V(نQÜr!I䔪¡X¶/̊¸땶7=۪X3݆؇³З^±�#Wչg𢸟�7µ¡E©k\rօ¹Gҩ폴We4öV؝½󦷏\0R􈎒!0ܱWݣy¢CPʣ!�|Àgn´ۮ\r㰌9¿Aݸ¶۶ø^׸vÁl\"b쐼yHYȲ깅0ҟ.ý:y꬞ᚶ:²ؿ·nû\0Qµ7Ḣkü<\0򩦹¸譮B軳Á;ֹ¤򣠗³ʐ ϦÁ/n奇w�A׵ö¥AÁ0yu)¦­¬kLƹtkۜ0ø;ɤ=%m.öׅc5¨f쒯¸*׀4ݠҖ¼cÿƸ܆|\"맳򨸜\ڦ¸PNÁ𱻗ȁsfΔ~PʰHp\n~«>T_³ґOQϜ$ЖߞSpn1¤ʚ }=©L땼Jeuc¤©ءA|;ȓN󎄭º��R¦§ͳ ΉÁú.¬¤2Ъ腑ª`RE魞iP1&¸ވ(²\$Ѓ͙­5Ḙø·axh@у=Ʋ⠫>`þעЗ¯\r!b´𲎀ö2pø(=¡ݘø!es¯X4G򄛈hc 퍑S.мYjH𺋂ӖÀ 0柁j䜮f\r¥́Do𥸘\\1ÿҍI`(Һ! -3=0䔍 蓬Sø¼ӂgWe5¥𺜈(h©֤岜ӫKiʀY.¥ᒌȜ$@sѱEI&烄fSR}±Ųڽ?x\"¢@ng¬÷ÀPI\\U<��"E0t8Y齂`=£>Q񴃂k ¬¸+p`þ(8/N´qSKõr¯뿩*[JùRJY±&uĊ¢7¡¤³úؑ#ԾXû뿁P򃄁D򜤔قÁõY¬´<镣µX[½d«d庥읁a\$¸ΠüW稯ɂ趡+eYIw=9iٻq\r\nÿر賁xڰ]Q©<÷zI9~W役RDKI6ی큞Cz\"0NWWzH4½ xgûת¯x&چ¿aӃ萜\鸃ན^ԓ´þKHx¨ٓ0腃҂ɚ㧘µk,񎼄R ~	ౌ󐗎yºSzú¨6\0D	桬𘏆hs|.õ򽉂x}/µN烈ü'R公n'|so8r壅t覩ȡ¨\0°5P򝖠dẘǆ̕q³¹5(XµHp|K¬2`µ]FU~!ʘ=堊|򬵰ꂜ\ ¾C¨o甶e╙C}*¨¥f¢#shpᏵ拚³®mZx০n~v)DH4祆vɖªõby򶔊ǌ¥,´��y,̫֞2¹􍺞÷¥ K2¢xo	 ·2ѠIùa½hõ~ ­cꀥjõ6­שÿ]¦Ԙ¢5׍dG׊Eδˆ'NὒVНɜ@оࢿ^匚öp:k˱StTԙFF´`´¾`ø򻻔ﭩ4÷7āpcP򘷶õVÀ칂ىLt	M¶µ­һöC©l±±n47sɐL¬!񹔻l a°Á½!pG%܏)<Á·2*伌9rVɸ\\©³]tn\r<ėް£vJ栱I㩠1½Ys{uHհ?뛖®ǎUo䁟r`SÿCcv˳J釣§µõԻ=ϣ-H/À®ر'E° |߻\r};ø>þx貛ふ5B¸*\0¹ଈM³©\0̻HU·ö烢¹W廳ɹB'ͼǶ󛓴s¾٭yÿ»瀙ﻯQ߄ü>?/<µK@  À¨B|aH\"¾ R	ှ~@BhEL\$𮛊°Sa \"Ђ0솥`b\0üÀ@\n`=ҭnڹ.*̔菘´n¯<jO¦lM\"mRʃΰ/±¦*肙Ĕû _E4莌úϸܰ玼R0*񯛃֊Bo>S%\$ª ȌN¸<΅ξ𹆯7\n§̷툌޴,鯢󰎺°¶쐬퐴퐜"l&T테05nùü㯩вø𶤮邐蹆֣Bp𲰋ρ\nԧPِ݋Ю-,攱÷Àָ3\r/p°P߄ b¾醁%mШΐ2?P°񀳰÷0(ö/gpz÷0茠܌ѧυ𗁏\\嬳q򱾸pú@\\©ªu댌@ ؜$Ne°Q¦ú猨0(A(¦mcL'`Bh\r-!͢`񛞂k`ʋ ¶٤þ`N˰§	§ЯnNנú»D\0ڀ~¦ĆÀ`K⃂] לr¨|®©Àʾc˶iԙ带㢜r4 ,v̜0ދQ؉ NÀ񘯏쭴© q©'ª tr\$°䮰춥ꥬyMb؊(ⓩL')¶ޯL²MIs {&񠋈@d׬¶wf0ɭx§ֶ§ö~3¯X½h0\"䒻DΌ+򁬏\$ b\$ǥ2VŌ¾̠Q\"¢%¦քR©FVÀNy+F\n ¤	 %fz½+1Z¿ĘMɾÀR%@ڝ6\"좎5.²䄜0无Ď¤d¾4Ŕ'l|9.#`䵥憀¶أj6뎤ö ¶čvڥ¤\rh\rȳ7©\"@¾\\DŰi8cq8Ğ	\0ֶbL. ¶\rdTb@E 蠜\2`P( B'㈋¶º0 ¶/഼3ú³þ즒.Ss+-¢ᠣAi4K}麓¬¾ກ\0O9,©B䀀CC'B@N=ϻ7S¿<3ǄIڍW7³҅Dö\rŨ°v¹@½DȺ9 񂬔~\r虜5z^r!񽉍¥텳B螦\0eTK!ÁKUH´��¨2i%<=І؞ úçٸr7s҆ǥN³»E@vól5\rpǜ$­@¤ Έ£PÀԜr✤=Ѕ%4䮘\\XdúԌz٬~Oস끺m\" &g5Qn½(ൕ5&rs N\r9잔®IY63g6µ]Qsvâ/O 򼈎¨@ʹ§^ur\"UvIõ{V-MVuOD h`5túɜ0ԇӔõ,	(ߪ®qRG.l6[S0@ѥ´¶C}T7擸5mY뉕ú)õ8ۃú¹r⻴ئ)´M+ÿ4	À ɇ4õ̼©Ϊ1ԚJ`׉5X,¬L\07T\rx­爑dR*އ¦ۊЦ\rص52À𗋭Cm1SR骔`N¢e@'Ʀ*ª*`ø>£\0|¢ퟦ�E,¨ag.ˣupƃ9`B¸ªaa¶¨ްꠤmҠR~\0ö֧-cmO´񱈜reINQN�\rþnq܏¶��n´Sn�wƃ¦\r ]a¶¤-ϡ*¬¯\\5Wpv^ OV`AFÀ脇3#82põH'J'nM('M=j¹k´ZbBn<<狠\0¾fe¤:\0싨úN´õ¼vퟯ��闱¶ލH(Qg􂂵Ʉy< �\\¥c\\򳬵֋q0­~¢i~뎌e°Ѷ¢ҪѾöȠù~ ƍ؋mٿғ}Wؘ\r慀Ԍ\"i¤\$B㿲ᣤg䞵b?ʶ!w֓+xl1`¾`ށ	s؟ 귋¨ˮǶCnhEd Qөd\"6µ´`¨\"&f옸µ(\"ⲪQz眤ě0%±0lw uך>w류±µ%»w²Z̜"-ÿu�싷󕤓Yg±þ>x\\˭פ¼࠭v\\ýx^'M	Pù̝YP됬ݐù)8û%C§@؄F ̒\r@Ĝ0¼\\ྰNⱮS\$¹YIՃI ة÷>xP͸͒¹:ͷ򽖈T,⧌칙ͱБ2͌¼\r¬񚌒Ϥ¼­ΔЙрОҹF¸`ùOf¸Ow¾\\hؽ¸}SjGGWA팇£RJ\$JP+ӷ§L¯v,ә(̵ǚP째ޔڦz+ ޡj˘7 ͷ¦-vA÷Ϩ ڞ9󔙶ODùZºC¡mù`OÀR¢yӒϞ쐡뇶zs¥G\$IhY񖕧ٽ58¼¹xFøõ§ø«¨Y9¨©iݍ8´UöCڛњe««Zq¥uA«±§¿ùو̹!°½:ړ켏øb0{\rúQh`Md7Ỳ ۲8ֈ`%Ƹº㻭ÁlʃXk³H¡Ӟټ\0ν؆`ShխXʖ»\r·揻y¸X¸¤¸ :w7±·ڄ׮Ʃ²ңû/¢:4(Mº;­¢øc»D£z;¼Z3¼»¹£½¢ҝ¶砛ؿ.ªń¹\roؙbO쨞`Ϻ¶|ª놗÷ۉ/ٿX׎]¼|ü^!%Xٽ³8Ճܜ$̻ġz¹T媸K·¹-~² 8X)<!֨y«9ø¯ú·ª:ûĠىFú®xz+U຃ÿ¶¼úA¬E; ª'ťc­ùۅYߪ³ªüw¯<{¦õ9úøV:ý`͊ʇ<ءÁüGءǙõ¥\0儚ü÷U Zq\nmx¿)_¿}YǏ韩z­ ù­y\rҙѬۚ3L̙ªљ²Ι¸ϻ>Mҭ	M͉ú)¸P\0u8 S!Z{Y¼䔜9θَúfV3oõOϼþE½РCЭ񐠿¿XU¿սҬw©0´}©­Ҍ͙ݷY3ӬӔÁ4˝G݊Й¾&¹äنͭ(َʭA֜V=f|Ҕؘ@E/ߥ\0r}þޮnnǜ0䇌y¡¶<+Ӡö_¨|ʫ#AŶ\"C[y֚EW³鲗²f(\0澤Л>À) ހ̟ȕ났U؜\ù#ýe½*r`܎Y ۪£=aþ\\֦^g4ޭü�ýإ#讔^°|ނ¡QXNܧ漉義礜0rƉþ�^Y薣斩龫쾥׾˪ގޔFÀW^蒥¾ݒ\$+ՍPիY*u¢~֖,¶ō՗W͂hhGÀ¿K´\\C¬鿷HmZ抖ÀSZ_U֥朲­õb)õ´gg	q񻭶ö@@Ս볐݈΅t䜲J°ǎ۔ӗ7sÿ¤¹¯U¬K_1巴¾j&S傩\0، &\r¬ Ҍ`ø:µj҆À~=T̪¢¾gϤ¾�û枨Þ헕𷗝½믛{ùB¡ɨ杯|͖ȧ񪯈d\\ޖSɗﭹ¡ÁG`u­̱Սفʿɽ§3}򑗂\$qI򭂾°G=ꯖz¬\0_põ§´!tᄲ{ے^Z&§ü	uӚX¸ö1@ÀЏG{䐞øõЬ¾	NIҤӂ��=0ÀBu82Ɠ\"񶸮Qpj믋v\r<®Չ¶U¥\0.ù¯Ռ¨EÁM\n8VүQ\\`?༌6¬ª̽\rl±¥¨¶±쀠\"øુõB2pu즜0凖낵¤\rj¥ª0Vˁ¼µ;v\0eH;ʇTJ¢ŶpH?/\\ȵ@!pp¸C¦ʑ+5\\+a8;\r(*³Tǆ¢;ɖO|¸^Ld&/¨񎉥Tȃ��G᩠j%Ǘ䇄Ԝٛڃġ4ɮii­ 4·󝀴Ɔ#5cõľ÷𗉕Z¢R񹒠@ँ\$I{zÿ胇充| ¼¦׉ª܀@=hCEöՈ¶, ,ZiµµKºàP¦|,g°z*ʖƱᅩAjknK\nºÀC\"J79À}4f¢*´4붵¶ê­׏«Q\\¡ޒcMᑜr{*۱j¯­譏lFm𴬙ō¨* `☹GÀDÀA-qqab´±1ª9RɈ¾Ţ¡g8+¬l/³¦䴹ń栨 ʀL\" 8ȭ谨Dc¿#ihc`Ǹ±¹A1\\ùuK(д!¶dҳ¢8ûΘ¾шц®4¢j;#¯Ø񯁘Às8Àƺ5,ucncFöNpPa8Ǉ8겎KĒцǱύګiȋ4A	£8TҨƲ6 ;*i㘋²%M݂JG² &탪1T\n4	-#讗%Ƙڇ¯'z#ʅ8󁘎A+@S.0ӗüõ቉`UºQ°Εdd\$)¤Ȫ]¼ͭT驣ƃ訹M*ퟥ�b+ù·ѽΑĹdt\0-¹Lü8\$¥\$¯ڼAɍ!뤏\$p@]d£¸ߦɍ+2E´y߈((_|ōdÀvU9!¥D	ѨúªW=򗆣ø駴bN;¡'£¡\0²O¡<LiAɘ РT¸£¡\0¾QɊ# }Ba(𯊵GB¼%-)ʨ򵄴¥~\0I数Pr+1©ª¤¤%51ĉL`ܞE'(/Ô¬£%Tɩ9˚OrT㝬ٿ˼ӡՉ¯/|À\$O𐀚 Љ®XN|±%,¹SK:]ha%¥ª)kʾP\0,·¥»'ఊ©:ҿ	䗕æ��V£0újهJM¡*x£􆐩¬jKȒ û¦\\\ru\rۨÐWԘ١F: kºퟑ�ƈNJP!Q2 'H *\0g T|΀ª~g`D,͏¾\0#¨	³;(\0 À ̵L��5'і`'´Φt(Lög󁙤Ϝ0য়ksi񸦂Ղød󭁸ºP\"Ng`O&ċ  X@²	£ͥsh��_܅sb¨fϵɋM>s3@Tݧ77À+򮓔dӧ5'6s\0\\ԧ\0O:ퟸ�@ P滻9ڍ¶pFπ78_l³9°\n¦)Rg³9@a痺iۜ0þvSD򧮂ú\0¸S݁ù\0øsõM\0Bȟ\0+Oq`§׾ل4	 T9ۧ7=°Mⶓ=qø'y;ৌF領) ϖwP·TӦ;\0¡O|?0)O־༟̾禄`#N´ \0¦ù>'Ϫ}ՠ¬ͧ>񀢾e	\0? *P3¡\\ÿ楀Ӎ��\r'¿C𐖈 O¡E\nMBʣкT;µ绽jPރ49¢¥繅z#NƉ٢΀FYҍ̄\\½\0CA QJTV��©­鷠\nvӰ@_®Q¸LْRc!VÀ|zҍ6¿KKюS£4¥ɜ$aIª|PA+¸.qKD-瓠®EvbCO>¡H¬ªٌ<ᭃ\r#䣌Pܘs⥺P֭20 =¥*ÀWL2dത \0ءù<	࢙°q\\pa@Rd ofKM ݓp ¡±§\0}ȶ񪺊\0у⩡2Ձ¦3\" )@\\*gӲM#!żÀɏXT\"`\n];S CΠBͣPӲ  [¨¬ɇ\$4pԦ¢\"¨ϩǎPާJ©\rE&8z԰ۀ>¨႓RÀݛi\\¨uD*vzºSć\$*´̝TZ¦\ndնiª¢+J¥D1I:=Аۃ䐀Ɋ͜"q@| p¦övjoT@SSəڦ쇀*ϖç8\n#之֠+`ɋ쓃!⨅:QΚj󉚼㧟Xђϰd磬%aX^OuG򢥇ø'󂰟\0{VúМ0°gQWxX暝o>ׁB'¡= 'L䩔v ՜071L¶ªkꔻBӒt§GՎ,܌ §RMZRWL©U֔	煋	ڗngl,T¢PȜ0­:§`*YtSW\\ࠜn耓_ȟK&[媵\\Ԥ\0ºÁU¯® þۈ\0^\rrC\\±;صEԻͿW%ຆ¶\n!PZ£ö̂wWW¤\0{ŷꢜ$򫁔[űhB ڐ\0ºƶ3.kw,l0öئ¹¶:¥5cвXüFõ*򭪓2ژꩃeeeیº·&±²}7««,2¾Á߯Ź]D\\¢G吒.g2ĸ\njÀ´][]э	dcԞ}{죔锘ޫxGٳ,¢\"ܺvQ픜\nۈ7ڟyu~ =BܪФQEs덙揵?Z¸FHü»N¼,ۄ_±R̜0Mh狯~­y|C»V©ª^5R³2%ݚ¤ǖs*lo{,[vB©µ1\0Ƭ>״쯺諚钙a\nᶡ논 Aؔ*Ãa¸2ݜ0a˫@z籜\70ࡍr©­ǩʐvfX؂°g8蚞6g1ÀN©o¤9%÷S׮t\\珈HȄ\rއ\n·𗞦ځ¡ö/°崭{2<Ҷ&Xi½_򕒬f«Śϖз\\tئퟏ�\0¦gb�JI,ˠ\n) .R«\nT-yDыx¡_ /綸:¥H®»W5^UQ5ѫsUZIUwfFù¶£¡R񇓜Ɣ鱽Dnz۶Fȏ´­\"\0\\󱜮vݷW¯++g2рµݓRKªC¶bÁL¥&=7񈘅\n	6¤ǜ0r )ü\n>g1¼ыބ8)Xo³÷ܤU;ĆP薌¯z዁Jö¿}㈻r+ѝdx\r-쇖%BA&\\;̥'񯭼za<nð =Ȟ=|µ8᧎@`6·¶teD𡩅}q_d9\n	A#}µڟ0¤í¾`廢Ꜳɲc>·篁~d߰L§?b%÷/}ûú\r¾þþ³;\0\\£`ցP\"-!ۮ!桙ߛϯö71̼@¬¼MþĂ*ùA«jn̖\"`jû·򊒳°°1 E§1m6¥ú/G܁_»­b s_zù£'~ΝL:߂朜ùÀ҇cFa-8()里©a񮦬ņÁFp¶Eÿø?¿XͰ)2³³֜")pi»ᴽ1ЂüªõË½±^¬¹pߘ5ҀP߭˿x򜋑wϯ·~@ɵw ¿󇱦¬JၝzÀjl𞤌	`P¸©ƃ§²\"/d{8¤þɧXQd)8Ė	qx3֖ەċņb{,1इqMŁ·>D\\X3g㢢gŤn퉄¢bᶸxЎҳ¢büdnU^!&|k	9=Yv±¿LqϜ\"Ǻ²ڄګ¼ӟwõŎ׹ܣᖾ꫻̴.򘼄޸n÷ª¾<qI|Hbùĕýr-µ䮶°,Є˺a۝^ڮ׆ȞFׄü[_̭÷¿ªú/Ozÿ8ٌøø£\0چ𻁖倬ɈWÁm´O(¢½¨W¥ɛ'@¯\$Ȱ􇒆Ct	>`tQBþ²÷Č¤lm̉8J⏒¶m%`ŞKp1³őġ¡ﲋӜ0ü=¢˹)2׬µ4±P·#ȳŲ҄-TQ÷>£Ԯ%\rܥٖҨ²®߫À¸6@ަ@y¾̞��𘈶¬ላþʖh)\"©R2E(̠³󭉂H¯0e.p³윤¨ѭ泷%¨)voɜ$/¬º+.팈ʷce;ўøʁd­Wy冶v¤@q 47Fy;ÀlY\nxJ姲ª󐝱lӽ́xʦ-ಝü*d}WA@.OMǯ;<ک\09Hlī\0¼\0ѦڕR9øیœѦЦ`cώC¨D䚬aϽܩ¦=h(öD@ЄGÀۨP¬W«f@𯂧ÿ@&H\0 À;軅ºN´-̺hoC¹9и4mM}l렺&4v½\r{Ezõ˴`?\rh{F«́n@ӥB'қѤf\0腲B#xZaӶ9=7`ʙ¨°ٗ\0Ⓛ򜜲ÿWý½¸³>jª\rى陹f¼!Ц¿À°ҫSÿ<e̔6橬Ϧº򔈔¦?|û¾jHY隵Sª9窰6KN§G}+긄ퟔ�)Dh舋Гøn[1¨g̊¥_ٽªvûؽU§ЉX\0ຫSA@R+üûVͭ?墓«傒̖jB򷓁榘p6j#Gy񔘍\0慒¯菘f6ûC@r½²⁍PᚗֹuødǦ÷צ@󯒝뼢zȡ&²֥|hޢY؛G𯱀جy(Ŭֆ@߼,»dᙵ۟«0leV@޷=}^[+ىWR±后ଙ¶\nK´Q\0¥\\0¤0ގCµ|씻i»,ٶ0ͳ]|폧¡ڙ٩¶²q'쏀{тUD`eLIP暡Wö¯�û_ك𢇄ýKi󩸿¸Q;ժ¢zɩ¹8 2奺b´¨М\0/%ùcªýv󌡶S^\0[񘔈!ۚҍ񖚂ཨ٠F\0D@ƹ2E\0􇚌A µ՚IDFù¤nf³M©t�i·Ցb±J`ꏂ\n©¯zǗV&¡û(昇À(P瞙GH嵮T ûݜ¡G񉌇-\$懃@=﬐L\\ø½¸­¼o/÷T釪Hꃷ!ϺúXMz¡ǋ㈙û:wK¦-;<|׻o\0uؘªdܿ\\½fF{ˆ¬IyܲöI/qݯǕ賢(ݓÿ@\n佁¨̍ெ­w¿	)	m̪ʄj3ûX¶Ծ÷⡜$¸@»Gƨ8oµܩ͓!o½䠻¯~It¹ª닼>Ҹr7\nmÖ|<¥¡?Eޞ7\0»9W𖦖k¾8⾻£슷%ºQ¸a܌s}ՁÁַ±ڶ1IǓMɝݯ´0N#o«N4ʷ􅡭χ#x׮淎^bmҶ浅¾P櫔\\ெ鈌𺵷¶ሷ³Br_¼-xöE3¥ÀN\0+)··ɭ{ޤ󬗼˥쭲»ۂ¿%´¡½󊟃¬i_¾pW¯ H,OぺLoܑQ7¡q<Fܐz/i/3v tư񄓬펮ps¬©ri­k²¤W pƩ:ºpÁº½ȕf°¡}\r8gAü +:\0lȮt𝑜¨ԓ]å©§ν\$/dt¦¹P-ޕ¨،u¤D¹÷ʾܶ倐øWT®e飃ü¼½\n'°:e.÷?2øӭGrS?<ඒù9¤妜똜{÷2R¹¥­󇫺RӯX8%ǡlZ␂t,¶M¬ۓf䋷ÁȖ򸟚>jsl\\ݧ77𧎝.󍶝ȟ:^ePöpL¸K#@õ`\\GLퟋ�¡%@էzQm¾�ꩭ\$ºö3❀􌡮\rV\nՉݜ$화痰2õԎýwÁ% ˓ԬS¥]¸­𳍔ີ󶤣\r¯3Tၻ7e(ؿvOr/õ訥.&0ºû̮s#º媯Y>#)ù­¦c_Rшq¯§и\n욚Rjp5ӑr᮰Q¡з?½ =煮\0x*¯-Áƻø.û6F?#1G|Ŝ0.x#*nc>´`8k£%`S¼S၌\r¢񰫼7°\n¸୺þڜ[ĞFõ;膏ퟱ�\0T۫柹û_ªnv辸´ÿ¸2ڼµ@훨޾Wژ쩜,mb*v\"Ǜ0yvª¤p [<҆#篮!+{ ²7v0ùO®ş7귻¨@̸\r5~\\F̰Nºyֺ³}	²o/샐»U󞑻濸{®#Zϣöu魧턖Á¬·xkӐ7'{de¶©y\\GմýúԦļ 1¼  d\$@9\$M¿w6��_dـö\\= )wgsPȩ=ݿþ¶=s뿞 töȺ§g¿h�­¼˩njz'¸{Yַiù_뽗촹\r¼3٫\"g¨9󘞎r膸:y÷§¿ຠû%��며僵Àe¬׿3 \nwdӒǚ`pXR`��夃պ¹eÁjDjUtמ¼Áw±>̟ª«+±X=·ԞBƮÿ\rϋþb¾ߞSd4ö6I÷ϐ´,^9¹®ýáu,}0ş·x¶÷ʑ᜜Ǆ!ü͊Ӻk뗅>`1蠈:°ºPퟜ\To׉\$F&ýT i0Ⱥý°_nيÿ÷/[	~÷À۷pÁ󁟃>õºn地¤Àf׀؉yǽ󤛏?®D쇜0¹©4Z\0ۍ=õށ,¨!ÀSչ¨I\"ߑ²ªΆ]½õ_%ùk߹򯟜r霰䜛;1¶v?t¯T®䜢^ªþ8mêe瞓7!ȷ俫ܾ@°F	³xÁføБ^#󷧼qûԟü¤Ě¹䈷٦Տ¬E¿둮볏©ºř£ɯþ��謸ʋqN؇þÿyĤϰ[nw3:ݫ´ѻL»ہ򵜰ý;*õP6)½*Ȏ²¿Ҩ¦橐´=Ԝ$򢜲V%\nRRA}Ԇ\"ÀfªP = 1شɽ&:>\$^RMpö§\$IL\0ö1񨰥ҲÁ}Ղa	HÀz	¥²	£@ 1ږɽ&L	A	@̔ºT«<\r©^ÀjÀx.°%¬p.0Q¸£9\nø(உۀ¦`(슜0ʦ>¯ÁԜn÷!@2DÀ֒´\$iE\0f¸@v\n*`£¼dW᳀\\ༀ>H *©@ª¥񑇪ڐ;À|<|!IÀȄ0#@=)҇kºX0WÀ®1؝Z@敬D@ (P_@t¡۰)AÁ/°c³0-Am䋉kÁ	P2ÀჂQ𵀔1A%{±©©=\$PI=K(𰔁V¼°*@sAҢс܋¦7@¿Yǁ̕\$\0ÁՇ`.°9Á߆¤Ё\$ù¨Rp{¤ü¥PV<Ǉ��#\$ po¥°*dPNA<`Ĕ AG²𑀀TD\$©I¥QBLN©C©=\0¶偓ÿ[°<'A½	´ɩ%��\$BW𶥓	\nЙ쐌&°@}İA	\\㑁Ӄ`¹£t£ǭщ��'z[¨	`5Bº¤\$`ၴBS07A¥L悸	䜤所û ТBA	/悶U,,0\0ޖt¢2@׈l\r钂Ӈi/°¾Ќ.r¤䮉_X(rC\n̯¹B\n��AŃ!02C5<,ЇBш.ж<x𔀭tꝐWoďù N	Šֲ׀öJރІ@¬.þ<ЅᜮC¢؎`れl¹þ\\\"砡𸿄;`3^ \n@°'¸	C`( (е&¼\$䚪\rL:釾¼쟃°¦󛁪C펴<À<¦vQÀ0§\0CªºXH\0002^;]	~Ⱌ°򀣐üD1(5(�49ĆQ± čԅ1@ܻ녿D\$k䀓āD#DsDGD􉅦DJ±#ıTKQ .´¢ÁCüH©ĭ<Gq\0= !¡jC½aı 5	°ﲌúM±=Èɋ@O仭Ŝr@剅\"¡iqħO¯Ô49锃⫈𺄵¼?0ý葐ÿ2¨§AMD§F񷄑䅀7Ĝ\üF𴒷dRQ6DÒ¼N1ĚĖr³A+vE\r\\M±DᒌV1YūO񣄫ܙqXE|X@ヂ2ciD©lLQ^śYূꥐQoEY,\\1fE¯\n¦𡨃̝QdŝWљEɖ̆)#hװõĘ<|*d䅲x¶වރ;¦©G^㜢\$h%Jv\0֛X▆/¸+±¦w3GŎbӳqüFq¦5D9YªÜ0Á.@ø䏀69\nt+SEę*°³Դ\0DޟºńqS鄬¢x8±¤Ӝr|g@!_Ԑూퟜ�\$4ᷭ~Гø1¤n¿r¯\0*\0#E±°0?	'\0d¡ 	À(Hxنٛ \$ù\0 Hþ`(¦±¾¦xGDtH롒\nɁ ¬ƞFJJ|ᶠ>?)PBQF㜱΂\$`¬\0>/jД\r\\s㺉(+ nȏ!Fq^µÀ֬s1n2¢«dD1ׇx\0a/ǾFᴼݸHྴZ䜜À6Ő_쵺8񹃐?¶²4 7궜0¦\$y0«Z >\0ļƦGΐܳ,¥ǖ ý±^DS|s )򾘏ú5Àq1��LG#[ùü槈L¬i[½ H&H	܁҄1 S[&y°ˠ#G¸fퟮ�«[i\0#5Јiü+÷\$N(-Jfퟫ�&9.E¥\$Д 凂3!p񚆖\$¨0 톴��µ:eǬº͛0ɈɱϺ|¤E!¢ø\$,¡,K䊀; ʀ0¡l\\~\0¬L1Q£و\0%x@3ȁ˗ 2G`L[ÀܯB֊b 2dL\0腉Áߘ¹Į+r³Ȝn:Ȍ෈iüୌ\$lO텉4Xd\r)D1Բҵ)ٟQ¢WJ֨#±ꈰՀ<Àú%hrEüû QH¿\$ܳ#	ؙi¢W¢f¨gHa8!q̾܂D ý	^H𔱙9%̏ʜ0΀õHסЍ*JGiɜ"¥ߌ,Mɜ$´øŎƍ Y\"²II£X;ɟ)(磀s·񐤶\0؊/a'POﰈ𿜰!`2®ÀƱ̈,O񊖲²®ˀø-Ĝ::L²D~-Ѓ¢_Ɇ萒]HF 	򸹔⒲²²Hb*pɛ(뙲´B\rȔ;-(üGҎ,:ؼ¤ ֜0¬󜦆¦ʪD<fÁ­p¬@)(Ó򩲲(üOº¼ü[¯À¼茎谑2¥\0軴'􈎱P턧 ĕ®öɐ\nԕ\0úʵ%k]\0٨b H¦(øaHH# I@)ҙ\0ҩ+CþJ윲l¦oN\0ø̩ʾ¿¬©Á°	üng,0n򪄛+ûH\"I¶7\$¡Itªܲ <ء0H@ʥ+¤¬¤)؃d°!aT´rD«±rՀ٬p\rD`MGX޼­N=5 ZƯ><£P/섞g`˪+u򩥭%Ӧ⥀'L ʎԚҞ\nK.Q㔊橬Á˫.ءҝK½.ԧ¬gKº\"4¡\"݈(¡$y-ø )øW¸r/M²ûK¹/¾ýc/£R ,E Ñ¢A\0®\0HQ°BL\"Ҥ2/\$º%/��̯0¤¼2<L,ʌ¬s\r@	-󹢹Ƞ;􄲗\$N|Y.¤Æoﰄә̣%Ԇҫ˹+¼¾@ڏf ܲ¿^s/ꕮ½´ȏꌄBi%Mל0ײº\"W¿þ+A³!\$)󥄈Lª2¨9/0ˠt,̳3?'̪y̸C󦈋Fûޤ؀ħ\0T 呔؂6ªpȋ3򪲢ĐÀ栗ϛޫöđ#ؚ཈\n¾𕌷¡/¡ě·\\X	᪆ÀC㍍\$µùw²T\0򩴑U\nLkߏ{\"Dԡn��ۂ¼h>!?®G5[%¤򃴑_(kQL󾌖ȫI¨¼ᖉ\$,fȩ乧񇸏dڲǹ#ºslµ³ܮ²\0IҊKé\rͭ%09ͳ޷*·!­E9ĭrµ¼*i¯ܖ̎2Sķ\$|\0芁øa¥j[+£̩ρdK¨T #F8(¬aǤ§үI.,MtHرː!H)¼ѳ񒰎<cõoJꐔ¨︔¥j~ɡ,{üZ¤@\n?Á8q²҂Ω9ĎJ䜤牋ο8S´\0)GC9b󣜰鲦³Y%iv\r©¼)0Ű&	Tªܑ¾ތºG􇜘|§М0ǭq2·+d¾o©8L-D¶̈́ե<¬³t^s\nk;ĭ&o\"^8񺌊ݫP\"	黳[3Î㵰 ,9ӵ ,ϟücړؚr_±̀Ù؀c-<1ł\0ªφd°±e-,ߓӎ⨨Z§-򁃂eH-̇ÿ=±_-µӌ_-r# <M-BӢ͉>@\r㎤d؁ÿÀ:ȵ³̚ˮ@9ϭ¨\r녬`(81Մꃔ¸­^ˊ	P2\"¦+q-N«&Dⓛ&!9\n��Á¤HT\r|Á§󂪴I粣Y°Z;½w.ĚÁµMҐ¼b·ܤ»K於sɄ;@«(.Jþݭa\n\0ILsb\$։Ɏ\nɈ⳯	_3󸐵-t��/-pC¯¥;AÁ´³)��õ?Sù򫘜$1Դ¨賑Ƅ˖\r]\0002\\\0ȓü£À֌¥)ȓ\$¾2\\À2MͲ«Ѹ(¼ȀېG2ӄw>dȍ1PӸms!W(¸kQQEHTϜ0\ny ԅٓpȡ'ÿGB«'򙼳ꈈT|u-׌僘B񛋰\$DvŸ¦ퟯ�˰A=¢W.l;!,B»P鯽뮋bɁjҌ/?¼»û��ዀ«5T[Qp·d»[D¼úMQt?ݓ´^ƍEԑQ[D𿔟QL)-:QEUÀ!¯CF\\ůnJ½Lΰ̕¡»ѐilQĈlȮ��rч\$ňrµTyƍF퓔{QÀ4gQCG(9xQF=ÿѵG\\±��H`7ǭĬʌ-!񬒜$À³ûҠ8%TR,R\$lԍmCЅÀ¤Xd떒F%\$ ̂T̂ɋ ѧ2VT!dÀ4䨫T=²իÀ4Қŀ %ߊ~ퟥ�­TÀ¯¥+R;¨80\rŜ䈃ˌ^QZXQԶ븐}¨ŉ9\"dewK8;tĆ]-JO��a¤Z\$`~񜕣V±\n-§0¢AKÁapւ%X\0림ȩ8ۚ ;Ϟ0A?IÀ¦أ\0㇠¨M3³ ɄFü¢HĔ\r񒍵'HQHȌú\rx򖃞£3ùM M̃Do0\$āꉰ��R;잰I?!(DRͰ¯(	ڇºO®󂖳\0\r+£ȠRLø`üǑIü󄚐Tr½Q\0#諀 Ǐ(aӃЏ` mjÁÁøùÀA̞Ĝ0>\0F%)]HrƶÁՅꝠɂ­\r쳰f4ٖ&\rRퟅ�ꜢRퟅ�dú޿\n`Ïğx\n@ ¦r?C@(\n麔4\n@জ0@@	r朜nx©ʜnd?\nԜ0§P򺕙OȂ\0\\ࠓö (\nƧ悠DHdТÀaRD)að1ZیF짜0®\rO¸´üӶ⛡KR÷PE@P´TU@ɔT}Ag!¥¸@ C©_vûs­ji¤F濘ý \" ?*0񼦋g!õ%Pm>НӱOտ.Ӿ=,,퟽�e	¹bȜ0^쨕9ƱO}uJøʪKԟSJJP/Ԩ	劵ET³TeKÿT¿7鬜0Y¸'MTߎP´úU)\n¢ZµMԖBȜ0ո	嗐»¥AWkÕePõ]ĞªU0魪ȹÀ퟉�ퟖ�ը3\\;¬ǆ=VYõN嘴=õh喝Z±%V͛`6U»,A!M3⩰­´ݞ`9F(ªÁռ¿W¡x處ûWiÀ<´©Wڜ5NH­^­ծաܖXaH\0ɿebcZÀ·XC¦µ֗XЂÀ3¼ýdUV+\rdT-gXٹµV\\dIVD¥iµzYXSH萖dA¨˗ըg«֏Ԣڀ緈9+\0«օZxõƽjc@V«X婕­\0[X¨P¯뚭j얉Zª£5³Z¨:3<Vem؊ݒXý[Oh<U¼L[­hÁM¤\0Oõ֦öHmu§,j/cr֕+(\ru񜜜raf6VZ°ɖ큦@7\0[X½,4Ӗ\"Á銝?¨USZՃL«Ȗ`\roUʈꖬ򧽜nM<r`¢G¤ퟂ�\r]K 䗬P襃75߅&-ø\rÀ<֠+s[ⷃj§㮕܏Wթ\0Ş¥kʅø󝲏Z]%^ <WÁ^ջ뮇5f1Wɸ:ؗZ¶À&9^µwq:W^R½uúׁ]©75üWy_ܐř׏`Zþ5`0[Uüׯ_ٖiµЗl¬աױ`=V׾¾쪀À?+_³*6X2BߌC=G򘀬Vص^X񡼘`Ņ֒E\rBx!ͲY ¬¶´/^=y6׋_ٶ4`my6Dhgi¥z)5C\nݟ¸J֛bv\"ؤ֝_­v.ح\\¥[C眮܎!%5χ0e}AM	(g򅱢2dö &9a(\"QFέc¢¨6\$Xe¹ö\nõjÀؖ��I½ sփVE`⾈|kG©ù5tr¿d°VLµZȶNY\"Erú÷eÀ3򜲀 #½XUTZ]ݥ\$̂^\rsքW施y?ء`❖V¡a̨؊rüX^]ÿׅf%e¦wcxCRSX+fwVi5bU@2Xxõ6;Y]øhvpي,¡Tٍfɝě]vlٚԇj~¹	g\r^ه^]!jX㦽~öqYє2\$\0¨þ ¬̵ٱg͛K®טm}ּYõhyv\0ƨ}\0007Iۀ+ͯ,¢\0ýؐ 0fUᶙޘ©ڬ	U£6[h뽁jڀ¥׽cձֳc³ù쮅5h°񊜮5d`X絋Böڳ5ý¦`ګaTVQxڅiüP֗aBڣhõ¦օ\$6m|¯Y&µ¥}®j?J~ֿ73QէZh«党ǘ൘eh\r§¾7-ymWа�Ѵ坰i®YOgeɗ\0IõDUȴEZ蕏c(͵k\0ۖųwD9W³.톘k
>\rͿ.\0掭¶Ζ¢ùõ}`<dGýZl赶¸ܬm]¤ښ	Wmµԕ超Adڲü¡ü¡¼.ݐ¶Zmŵ¶ଁm¨[#譏ö	1d=ºV؃sn­t6=ޡõ»3[§nýµSVץºŵ㛕me¶뎧^ֈ¯kuv皽n¾6𛍯­¥`םó¶÷a-¸ùٲ戅¢L[couZø[ovú؊\r¿@䛸>q6	ܜnè МUgwܫpeµYymݿ	۽nłא\\U]Հ֛\\\\ݵĶ۴䥃c5۽fdþUz9-pȜrמ?g}J֏B\rȎ`܉r5Ƿ%ɡbùö²VٯmÀ·܏3v·)ےսd%֝r͵w¿*axZ¡qV7)U鲽ʖö\\ǥ冢ƴ^vj/a>¾Gsu_W8\\Ûh>÷\\˱²½f[mb=ŷ¼'X[!Wû1FV³¶͛9Wm]񏅵r퍵`±ײ½ηmc² /񵽏¿sý˗:]Grܥ4¾ƙVi¤BAª½o҇ٷZ2A°ƳKõ>±ª>軠)\n²i#&{Q`±@%)rµNꠏ¸>𼗣V\"q\n]vڅn\neۀ\$¦ޛܨB¬¡\0集µv%۠̅ʾ舩¸§ö@	 °]=3\0*«R-ي«T?\n@#侸	 (]؂ݗr]嶅ץ¥wհ÷hxx`)£Qhÿ)Gþ +1vM⊲°	醀¤;\\©ø?¸0ªԏºȋ@\"ݴ૓*>ø颜0¾gµ(>-E@!»vݷv݆(üu@^mxŤާvm⠑]¥v 	ɾ^?\"qWm^£v`,7ªސX׿&rJp­^z�ޥx¥ٷ«¬ވ	ࡀ3{W­ޝv՛eݾ£-ܗ\0t-߀\$¢۷²vÀ\n ʂDI鞏wPi¦ŭW®¿vա\0(&₈\n@,އx]ݗ½^ټ◀ށwؕ7¹\0WŜ׃^W}5󷌦◨ü¯ޭ|ޠ&]빝ߗ¸]ø՟ׯކ?́ûxU郰޽{uõw±^/}E㷍ߟxõ緐¬\nE룰¦{yE嗖­yu朰ҿ]]Żýܷeއ|²j¡­_΂󗥟?}\r��S¸­֞Wޮ}緝y _՛¥۷¢ߦ-��Q}ʫ7c^1yŧIýϹöW¡`{®÷¤ޙt·mTSQXû·oF*·_)Qªg՜^Ђp\\©蜰Ň8¬½ڀ&´? þ׭_õ򍪀¨	©`~?¨\n÷m'£|ýױԮ=٢UH7탱ј.'ⲂU`\r㘡އ{¥'x&ޱ=ETa쩨̫Μr¸!\0«|]ÿ8ଁFTUúø`鼢oĊޕýޗ©y\0ĄT��ÿG]vƑס]¸\\ª&­>뀥v ü8Fࡄ]DW»GcPퟦ�ᏁüUù­߳D^|]CjԞ(ÿɨݙ}z¬㽦p^雔7\nx\$a?�ބҩ\$?þ& \"މ¦xm­𽘬ᅅ üW͡½}~a5ퟥ�Р°ü꿏ƂHCùCƿ@ý@(aXja¶õޯv® 8}­{*¸8✮.\r»)¶8m􏙿c𢗁f÷ȡՈݵ䊀}\$\n¼ⵈżr\0\0¯2jɬ&zȘ#蟽C·¬§ܭ@¿Ƀö*욅䘻_fø*+Z«n\0\$᱑P5\"ݐ@0\\©�@Mر¹⚗.8«b|®¸­˕{ʡv\n¶·b⥯ýøaϾ撸i燾õ>ԯ>8¹bՇn#Wܦ±PüҢ§𼸾§=|zkjTú*mø⺚ üeԻ0¨õԊ¡þx&]煸þ )K傀\\¸_A~Ѕw¡c?Ъ؎Ʃ븓엨ÿwДqú²Ђ­6㳍~6邟`鸣]ȅxq`µ-¸I⠌&6آᙈ暘ԣzu帕cw&-❆®ءݤ õÀ?V3ث⪭@\n¦6v\0¨«μÀ«ד¾xõ\0V¸\n¸£c있	)ø¦ox6>ت§\rձø÷cᎨ\nx¡οªz9\0䕐.øû`#=ù	bE}>À+߀rxء緰ý򥠬0øa¨& ׌䔫ƃ]ي⟼,|\n̢LՅU>&o2¬·bª֝N)X	\04|À&kWܕbzXŬדᖜnf÷vdIw^&µޏ¸£ĝöஂa\0!þD׏&񹲡wX	÷ǤҠE넹ݻu蹷ᑃ��㆜n¶.xϤÿ-C١¡P0µ]:²s«-|\"晡`tX@'ி.R`&·¬?g¡rߏ}C·cKМn  ↁ^SM뀯w¸\\·ÏÁ\"k՛`鹎8 /§¦? þ7¬´¬µ偃މÀaꂠ\nܢԯ`	ણBöX¹c➛Y@«&{wx	ɶǾ?ݸ>_ö·򹥡v[µ\\淜dƚ>ꘫSö4 \$§Ɲùe¿ޞ!­e恘ü&­S(dF]¥ ­:µ*Տ}ziߥ諘џ񺡲÷\0¨úrC񦣶­窟ᙘ½빌Աweᱹᯀnù浃¶d4_f(«^eYFʂcɀ恗𰩡ÀöX5`Ҟ.¦«Un`ު­<¾D%N颧¥6 #]ѝY \$Gbm能bb}ꏦ±C㰡0üUTõ­x#`­kzªԧ÷֭)蔻^m*ߑN ᳇(	øf-򮣴c¶o򹐜nȋ㺓)öpýǛʧ	«U齜0E¾pNq霦°5٠ ු\ni  ³{¾㽡߶.rJߓ,?FuX瘞zpü툨ýRɖ?(ÿy\"ԵS¸y±gH\r蕿«TªÀ#»SЈɏgs	㳜0i)*\0qo¹§\0¥VzÀ\"䁉jXe>ý񸚨b?b´ªw0	\0/㹷µ䉷(br\$ˏ՜¾qٷ)rc󧜲~䁚~a»y5ꗓ(b¢\n®ɟf׌2hsϦzh`ތ¦y顧¦¹¢²ذ'/G`\"'䠚*=]Žʂ蹞jµY<Ʀ­5E@+甌®}ö⊿\0ø5\0¿&Hy˦o¡¥ý6jIý\0λXg%¦~9ŧûxgº\$hSz١ȁöùP呀.X^zɡp£sʖ'瓺µ·ÿ¨~Nhz3GÁ£VD &'ڢ⾴[裊®ú·²ºݣø?õ��ջµbĿ&·䞸­Vø´ʂ&󏉞HýشԻݏX´㓭𠜤§͌\\\n ɕµ¸	닾  鉆¶PY̨q¹ VixȔEPùߜ"N7Ȩ񶖨÷dExڪ·»^֗ػϥöC@)dZ þڭþ/¹φú`ø[桚uøºh팽㺚ý|^	À饃Pþy++¦eF¹��F.¹ԦzRµU;+Z#PY·ݺ ýöⵒþW⪏|.UٖhUwö1*񑽹*¼§噂pùaC¹®ùH)ù'p­j@>eɚ#󦎃洷Ч#öjWՄއω鄑>㭙߶@\"@<Іõʝ⇦`d9̜»[!\0냞¨\r@VO.(b\rlǯ! ;@ڃ-4ųIªLЂT:©o\"aj̜r}&²ʃbT١Rʓ��͎铒ªL¨G`­À3jʤYjŞhzrℛJ\0IS¼¡>¯z\0۫˝\0­j뿦°¡ψ `.´lµi¥­±؆{C:⍒̙�0003͒j· ©W泺ʜ§­úϫ3@]­3ª*ѥ<Ỗc⍘A[¥\0y@<§ ?yCþ¦㚠\n`++O¾|ù¦böXþ¸c^	~Ny)ú頚Љ·磭Նআµ¥zXネ&¸	k±^QYJcI^Nº﫵N4³`þ¼ؙ붗*}㺫¯}»:÷kѺ%ٷЫ	ù¦¯Riö]µ¡巊+,µC@Øõоሊ0農\0擐UTϥ ¼vø\0?ۜnj±FIͲ. HжꈒỴ£ɨ]蜢°앣¤ۖ)!U)¥!dꠤ\0휢 Ԍ쒔؊¶¹䪆$Á²K,ӴΥ⑵±²Ʋ¿MӺü«P#%R	01®Ț]+ø8۴±FŃǀàdؙ¼<S)Cü꛹µº񱑬Ä¶r͓أ\n+ 4kK8��K!ʻ\"*ྜ0번5{%휤µثA³Ƽ§A9±ćTǾüTr\0녏µ\0asƁm֛hȕ񊇯fэO`mö	Ƥ	¬tCz><tD&¢\rޘ´Ǎ@ǒur(m )ÀR.¹bQ¦Zµ£a)��»l/۫ۖ%W⿅F;¸8»ömد}vGaB|~򢭨+'��-¤s'�6¬򿄔P­Y· û\0¾¨IB²˱²½¤ͬ.8o[zR°1®ۛ1õ¸r\$Ԓ@\$S½·̱û\0ю!FۡX fݒVh6sʜ\¦P;ɛG(Jc1j>硄Ѷ\r®ہönwV¦ۚ°͓¹ٺ²\rºLٯĒm«黤\n˸ɺ=¹휲b¦뛛	_¸`J{¨#! ?3'�¶ߡ¢死�`hrZ77vಸ¸紽ATµ©xɮJthµ³«\nh°aV{Q¼袗ȅ4́²F,D𒃘:±i[Àhʾ`ûùÆ일ꥥ¥ｐ3σLƱ;IH¼El جdϴؐɦ/M\$X¤Āٞ3|򑎭!Us6쏡ÀCЈ#	6³ҡ´s{\"ɚЖﲅ¡cý!¤ݫ򞲟U©0V᭚�õ<e󈺲մ ܉؄¹¨dõ%C?F+ɳ¿<¹!,﻽ur¬[baxgʖ̯£%©㡆ãoКX8!F휤M :𘓭2ml¸珘¹ ;O»䣻÷#`mwSS>ꬲNk0ۇ@3Nƛ3I辻计FD𠟚&|C¤څImg¶֖6eʱʵ>վ񘄛k̪ۑj󂽯A­GOs°ûdp兑🚬ü'쬿t[Q΅ļ@	_¸%[I°Cc3眮3ª¤6t졬𫠨̄쌉mDXr¿ʵÔ¼3J}Ð\"[½ꚴ㻝CNL+q	û<\$ZX;̃ý¬΀胻I܁鯁.¬\0<ɞZ²V½r#\rÀ[	4\n훫µ`F풆\0ɬql\\ 񫩋#|Z쥋0K.\nIZ¼̠@܄Ќ®¼-񆌐d^񌖢1SªތúȄφH.¼iZì}ªZծ\r«ͦm뗬¯ھ\r:񫇬s̜0󇇜ö±곇W\0001\0ȼu񝬟\\xq﫭¦mZǂG6ȇ׿'b2rȿ5Ʋȇ@򁇯\"\\r\"o yq鉇!򢱟ǯ#¼k\0󭛧St녞γҲOK7%2c뿉n´À<봙'\\urfӁ\\'H\n김úWL=/Ný۵NmUwv݂NªEڭÀüy[G·P&2^»ۇ?M窦¾M8þ)ٳԂ©c²򔴃Y´uvk[E˺ \0ǿ¡ùøH9˸dرˑȠ0جhCBƲ£ˇ)½{˛J|»򠲛²܎Җ,K¶s	̻A1㵲ar¹˾ø+Vaz#܍W̨Л«ʭ˷4`)Ù5+00\$,쳅̰8;\"a	W󛲁ûª͛򂰯ϜnFƠ񌤠۠G0&΃͛ݠҳÇ2\\ſ؝\\ݳ	\\ÿ܎͒t/5µȌva ®N¨\"V	󧠜+bMpGϼ1󚑋(<õ󥋔Dü²sʟ=޳ϴ),vN¥¶��ÿ@²\\¤ֿ=üٹǏxaƌvͼִOԀ»е:±öo񏗁 Գf̏6s[>}J䜜5ν¼֌¥\r׹¤&G͜ǒ¸Ľ=GW교y;\\Ū˧H<©Ӿ7\0SGs-͟H䦄ûÀHѹͪ{ς\"B;ᬫ5a8T´؅¦W¥úZL .󒚐5ĴӠLWܘ׍\"紓M¼ᮖ܁½݉GM³󘚦ӷt₪Ţ°uӹc秃\nW¿*ý6ێԏ&گa¾\0¸/!˓£ნ\ёȡᾭ¿ӈ叶ºÿ©ù°��C¼ᆚ?O.ଁ@O=*¦@­헽ߵj@`½Ԅ|³Мr̰9â׎4ڗԹ@úõMϐ⸄�Q|@ȑϏ>]RKϕ+㚗ϧ[À󿐿\\|ùsÿbȜ��\\§KȲٔ軝yݴ`/^]N]tۦ ´󏧔ᄗ\$ׂf_qOPý9З\\`]C±A?F݃pj v%B¯b½��ŵ# qý4��Pݴ瘸Cк1T㥙oqtAG\\&/k2}żƌ°A65\"Wg,¸#ǧ ³Z]1ڂ¤ωõńûهS\nHk\\õ]ݧ=ث(=Cʯւ򒫝9ک◭24݉̱»آr«\\ւź¶öՎ픿kûw퍜̺´ǎt·gPヵ¢2u«H䝉	Í=ýº?٘¼ü2靘gkQ͈пl5õ֙ew8l_ܧn˶MrŶӷ9ܸ이󏙷s>a%DԝIEۿe}ltҨ{՝ٙ__Oc拶HýCֹ)÷m掵Ӎ}=ۜ0»@07B	_ϓ򽉜\ҲBݽ��퟿�e]nB°¯M-v´휲ªv\0ɝpMɕ̤ݯu!@w¿Έ&½񴓲ĴՖªﮕ­u¾\"_´@°ܩN њ¿oNW챞s\n奶QމNᬷ\"ݼ®½¥ؽ¤AjF/-ýw흴q}ü̥ޯx÷\"9=̍z3_Pü»Kڭ:²fN©­h¹­QsٟXSjøݓ¿ۍе¦%윔vùۥ»Aᩆي뢈FB¥;R½pGG²\00063ϸ@61·¥׈ǻ%۫LSᶉğ?+¼öϦĆIu7üùt皡t\r7³°º0艹ùؗi俐3¬@n½R @øm΄følg!܏A͆&v䭎ܣ𫏗^\$ҙ¬霡j(Ӳ·_Μ¸¤o῿󿛕§õ̲KäC÷񾳷¿I̤ѻ浝¾\0q۰ö֎ݻwjÁº[򄹷aö÷596#Ό櫶o]ῤ~a¦ܿԽ'V¿u􏴠@򗠘ǻG-Οü᪂eI:º²팑˄¹кe{yԬi:v«纒}`Ӊإ¼义7'½ø]]¿ܹyý_gI;��R	vݏ¡ބªx\rc@sk܏¡½§&+½O¡ۊn%赵rür¯#t­'򯏶4¬WV±U«ú=ϰ66§¥}kS9w¡ܔzb͹>Y\n÷¥ غb̈MÁUzeY2ºú篜\¾;³?¡ۨx4M¥ēz¸?>.穏¨>¬q\"U`úӉܘ7;2»*tެaa nüo%?¬Qõ¹ퟩ�z͜§¬K¼Ϋ러>´÷5筁Ǳ¸'*Àù俔>´>FơF􋑁\"¾Čm̯?<ÿøkשޞtZ|«~ɬf~®¾Mû%쿲ڽ潜r ¬t3<g\\yl|･ьԬX򙜜m̃'󗸁þ±\$©¥휜sCㄺ8D¾t\\«\$ö͍¯µ½põw²秭^ͥпݡöWÝξ괯>b§ͅ7÷;ø`[^焊	§¹}s{пþү¼>G\\󗽎嬮¨⻜\$򆶫֧A󜜘ϧ¾㸠§¾・O`[׮Ks¸W°k𵳙ûͪû¾ϯ¶{箟\0jeÁÿ®\0000﹂ߜ\mzp	gAÿ¹Ϸ½Eٟ½¹熀VN¯»⯓񏿟û񏈝󛚇ǟﮬhDλ٨*S¿z߲Žҷ¿+vҏ+|A򌅝񐶪ӤvgBfŰ鼇߃J´WQΧ²ǖ{󨏲ˢu1䪊¬\$ʟ!󿯡ü^ÿ!сgʃTա\\ݠԀʱ6ÿx{篑<¹û:~G?þ¤{7獜r¿KuM¹	X\\񶝩w·þʽ+)¸}SI��;󸈑¹J稒?HøԿ֟Iø҇ϞýBüvGtM§׳|ӗC\0¯p·gOR¶-Έ ߀ûq;^􊯽⏃IPʉސý\rΦ緳ù돯¶{!嗰õC´ݴėµℏ騌HuڐД8{Výv􇪏Hb1G¹k÷ߜn¨üCָKf􇑧ߪp§+׾Rõ¿ϿᾘtøLȠ2mU!¯с?K6O2~1÷©ۣ))g2a¯%\£wZeȿZ󹸥i_u?õᜣׂ\nġW6}ýYYڲýѡPş~=ü7\"¦+̸F½úg\\χ'øµbº&h皻꾡ӉϜк֯«þ¨\nD²_~¦ey򹻜0B넅򉚕җOU0\$Sz¥&ퟲ�U¼FMSrRt2𲑌/2¨e°Ҟµ,TÀMeț¼1«aeE龔&.PХ&헥ɡu·¤􃺓z¥½RT.ÿ̚P6Ùª¿~򖧯`þûü¿ퟍ�P𕘿\$ܝõ9D%햜$j忍TEY𸃤YիøBP𴔎𗣠˗Ƒ%'þ·IU˘ϸ៿º=³?惁G2£f·ú]63]°§ٛjöU򹙥ÿ÷\0ٚÿú¿ÿµs󳿴l2M²ÀXX¬Àt\0e©«ֹۖD´Yfp聆˛ɜ0=_ºѓ顟A@°¹\\蜰뛠	'V憪(µ¾͛@\"Z´𝩘¸Ѕ]Rc­ɔu|gÁՊ-SEL¢ݥ¥ À&ºڍ<㨈°\nP\0ft³ö0\"J¿Q?ׄþWûüTSO÷@ú¿ʿqþkþG2¯􃩿ӂ0)g\0µ`5몑«ǜ0T؅_䜰(ꌭ*ÀG\npÀz;¼JÀghꅦZ\\,¬ʖӂ넛􌬅4°Mչ3´˂,M\\J\n\0Oi¬ӯ􀺏停\0@0|ø\n ¾Ï¿Õvºݿyo%^믑¨)~~쥠ި҈C\n䧞ąv ͝ڙ±td²ÀɟõƐěÁröj¤£uF뵙516ԉµdEO˨ҀϤɃEøֶᙷ`cӃh\n´ö2,´W°sky9ªAùb2``ɩS\"8;ࠕ­¸ʕ\n`*̩��\0U®ӜRp)° b̅=ҧ¨ (	°Aa~ġٚؠ,8 0hÁA攫򠀱󩨍ö(\"«㘸A#^:ʸ®( ,]`°顕­»¨%`U?Á&\0V֍F樬X³_\05³;欥^XªÀ±er͙Bcz ª1Ϝ0W񗋶X)̕Y»AS¸ӥ³h KɃퟄ�´ñt򶰃`­Abj¤¦0V ÀoeLí£%Oퟝ�ANEh¸.⠒4¥a˅Ƈ,PVºÁ^»ͯf̪`½°Τ°¾ňԘH&cXsÁ8´¿x}ց̼Ỳԡގ¹Śؔ1Ţ¨ā¤ַPi`°*zY|X,Ъܜ\0+) X-°F؈1䟀?37ش°H ၺb@Ǳ¥4QP` ޲)b놕敐t სM+贰fþ1,^<󒢵¬򚠱4cЅ«»^FX쓚¯u\0с3-- i´h¾];Füd6@צÁ-S󧅅圲ɱahӑ\\eú߲ǫ0MøV	\rDɿe³vܗ88,#ު#_3��֘/򃫈d\"8Cɂ򺵭ňµøp繰^=7M{b֕+RF鍘ó0f4ѸFPۻ¤¾Va¾󽖚jqf˱1VబÀj4gg6À@zö¦-́ͅüTퟥ�0µ#AÀڌU­à٭²ݞL̑±;%L<¶\nG\nRPGFԥ%S¥umd嵪=v֩,A9ꗠ嶉¨C t£¶ɇH½st@~[5),ۆ8þcؼ\"m>Lj9��ǣlrõûm#[|Ŷ㋶ˉM¶llfၨط#䍣6Ipѵĭ;xHްv½Áύ¢lú#}´y󁜞±¥F¸õ+«ƀø޶a:\n¨U¬ ۺ©Zløټ\$i HløMK¥¨°­6~L7°2Vü͋ۇ«õ¦䰇͈ی¤Cĕ²tP̄¯Q炲vڱ\"묣5o&ᇆmӋµ·J¸1üΫ8¶ƳLƠ\rևG)ü11¸FHd0ɛG¹O朤k+¯¸b¯C·ªS »П\r@;x¢g¨Ԝn{T�%]ؠpõJ¸ԖǅШ¥.)¶¦ɦÀ\r 8Gέgܛ7⋚ܡ2\\RCb£Dɝ°÷« ¨Á6¦O\\­盲m锞rvpõ¾{϶¯0Ɠ¦9^-B¡¹΋t7z⽖+¥ܑؘ%¿;lHqCɭ8ȗ퍆ڷ=逳§-ߑø54rú 3·)̰§.G:܅9b9#A(䜞%vh¸Áøh4ˉ\0gK5죳gע¢=䈗oE뭄ڣot<õԩ!朰s\0񏎞¨vP鸶X֞ۖ쮐[Cµ\rฌэ¼(RY㊴ÿP鬕mÁÀꝠM©·¥RnZ灲´?熐ÿ[» ÿ²䇌ʕaj*ڄ¦➟˽@-Rû\r~»=ù?P۾õ\\õ䈛܇詀¾ՐD򜜕tBEy1	ŕ¾Ƌ訨£ӈ&>��¼B؆¯ɜ"ĩ^TdýlC±¡jD/N¤;©\nD+竆ûǎ²\n¢¼¢򜢨!HYtٔ1܄䮣©\"+:¿`3EHɜ")ďƘ2#E¤ЋO¦΅¨ߙM|ǅૢ7擮i\nC~ec¢:½oNT��F÷­歄¶@غ\$𔓊ɢ>D¯¹³<H奱#P¤D@?8cȥզ險15	bp¥i幜"PD9½刷x*6\\@k8õ8𝗺J\0᪁񽴦҃TAa¨Ự¨¾%º􋑯Έā\0ד\r\n@pЭn{<9ꅩ	&k&ˉ)⧂0]L籦4¨R\"hgsxĜ́ț±7ҢDϪ낖¤?'½I\"ZºRZ~ùþ&I򷓡yډD,_'sJ֦-~򲽐lN׉ѯ¢d@tKpBdD䭮]\"\0d񥹘»յµ3¶ꊞø\rÀ<QG1BUŐoРA􌐘¢킝w¥Ʌ©q𣈨¤¯¥\"7.s:(쒧;J@©yT¾匘`k粊¢¤꠸#ȧÀ·¢ <𜜊¬]^\"Jx¨p.w��H¨]_kûM啊*`¿ȫ/eމ8¿±\nȭ`Ϡ=¼ª)üUH­ KS¹µ°¦y̔¸¬\nÁM*CùR0溫¯­TÀ󕕐z<®QZ¢¾K¤÷v+є°±Z⃶(wÀkd+淴	cԑ¿0Sᘲञ»�±ݣj½T맹)¿-YмͽüYX´®џ>»tÿn:᧪+_¢#ñ̽5B,XȯÀbººCG駌X:򁔵qG`\n¤㳮𣉬ܭ̾݌Ѹ䝂VÝ¡°򃢷n㥴püAk&Aýa+ڧJҞLŢ%l_!ڥ햛(=ÀTM²ب ǳ¶õn:´N³ĘkIAĪĜ0􁨢bؗ¡oΔª:l¬F/ʽT§Cwx7׵tܚ³솂\r^󳘙ʭ[A½.mj=XT٘¦ÿβR°\n´0Yɠ\"Oׁµ¤bLdABx𸕫?#SO譡°t¨®°œ¶ȅ	§ `®F_º̋~ü¤n㸔§ќ0ު:ɹN\r%ÀRgÉÁ&D¸øy»Ʊw«Ňae¿CַD/Faؔh*´\0rGe}¬䃘f± mÀAtüIø짗ªÀ£R/FvZ؜\PLH΀~c:W-7ÿୋ¢ܦ½8-,hଠf¥-?{I8r')Շ𻥌rލ¼2Jp\r`»_z¬㭭¤j8ԪT↪{\r 鴜$V񁜮ߎ=­?[\r􊈒ŔmGîL湝÷¤5~\r′±sၤ鲵7)s7۳³ý筑\"Ą¦ùÁҒ;hÀ¤	PNڂ\$»⧐1&㢻z|ڙBxI+ϴL£Zs:z+{Իn^ڜ0h%ûۯW~\nz@ݜ"Àwڨ7p\"g;3^Àsz궭0,Y|µoAⳒH¯.{#xF^mt蕎#ȗiݞʶYs圮'{ط=Ƃ,޽mSԤͯH;(𦕀=Wi=޲<듩ݻ'ӑΜ"򹲸`Àõɼ(½1΄꼰¬󡏣鈩ǣ£㵈ꡅ«¼·[±ޭǔq{PϤqx驂ö;3H\r©ö«Ŝ0¥[̴羑f^ŇMüq»۲·°텕½pn6񪯫ʆˮ\\À㥺t¯\$6»݇摌U¶1lR÷°^lju;*dpF¦À퟊�B8ԏ҉^ƣz}ϼ\nȑ㝵(ûù©{䘇©,¦ø2ʧ\n±ꜚ«G}ø3Ӱ±꣘=̜n4轀R6±>Ǭu̡V=«بÑ¹ zx8*Q񩏪#䔔À	M%īǻѳ:¹ZA鄼؎1õb+Ǎ\\Uu\0ԧҮÀ¯\0q\r��«©t㳃¡V?Cǜ\ü^ŉēץ𾢕@e:0!kų.єºÁ[¦õ°ÀÀLJ8n퉴S󷑲`Fm3µ÷FH폡cÁ:{û­ù;ᨩﮔ¬6\"¤!cÁ)A*<|▆;û¹૚\0�}¢޹󤋸¿]¥Ō15(܃蚏j¸ ý턦䡾럐vB%(ҩƐ>,\$¦B¨\0##]	HT°d·&\nXM@ǟ2t圜¤E®!·䞸ħѶ䷠#¾󕹡«ד	0䷅XBݠ.ú¶\0M͘HF/ZO`;ʝT륿A­Gtvûº(̎r6ǚuإ͸(ȏj>¯5 r<´±£ӂH@Qy\"À솧yIO϶¸µ6nEaºùϳ\\뀫\\򵑚iԕƆ#xXûwˢ£C9Ȱ3#ʻ²7'꼢!wZþ¤¤Ҫ7õª©Ik&üᯥJϬU*gUb2ISZ`\0		.*i0þµüC{,©K?Hþ݃ ײ²4䴃IUC#EýZ©2^edk	øT§#iUJ໒7I!jCS'喒<ɶ?¨~ý⫡Ϛ¤ªŀ¬«G¤Щ0%؇տKý`_hr䧭D!nHퟙ�$¦􂅗®>IpPÁ?ː@�LWBEU®I5buZ(URÖ́+%VO\$ꜢHaեj¤¡ºº4`a`s\0≐W\$٢P6I*ғ\\⫦´°5tr0)#FAɦ[*mWy\$À®­ےÀ¶Ka㷄ҘQɣ&\"\$Yɭ뤐?CVB²򉤗BRa䌈v콦Ki�`«ן?gG#=ʄ֧¬WA*\0¨»±̔²}¤뉏0ö'<6°s䑁բAql֧𒜰'11,Һ¹5UO«.j̖¤Ba	ԂOĚ;fJ`a`3hҹ&胬«5\0'\0¿½¬µö«õήߟ:¿ݽõÿRo½¯ªÀA²¡տઉҞ'Q~\\i<,hױ¯䝟ࢌR´퐊Ӕd @K4Fh𺜮/MŸ4򇬕W£ׂ|ɽ;K싙·ÁYg熥­2°üퟐ�3fbͮP³2FeEٿ°琉ӉvK5钇񳤡䎼®3|Ҩ TüQ¦&?¡ü㱊g¨˽²¬ٱ0gʕ#?򼅏ø´*ցx­Q3֭pföb4M¨>ûւpWI³4*Ί񊃁0²(À¤,,´/\$˔ʐ£ۆeX4b4M|­l°Ὶ??cF񕳇֭�¹RҪKf�$ׇ�ɉ\\ (㗥/	°j	§û!Ɔ¬µ\n+LԱùA(ȍ¡ц5vf¥`8z9AꍄYHعi¤ Jø\"§ú	Q!¬«Y«TD¼©r@?t򂶭=	°hͩS®T@36m,\r项 ¦̭@¶l¬婄ī8L©##£W\n8	[XxR雁¶§HlЁ­¥᠏JېûV DR¨¼β¤+,:ծ!ף©kª'ۓø¦ΐºE9r%¤4§®¼¢µ¤EuL򍮠47eö\\«ʍHúu5չ{؉ֈtG㻎셙°(ÉYHa98£we:ތTZ3zS崃÷䋦rڟ-STWÁҢ{A§;CI¶7褷11Dޘ汷r¹ωԾ59Á覴⷏wöfFLH¸Ӄ	<Ŝ$N8ûI⊢÷	·¢l~V&YLcpHS3)@2¯ø쁁).R㊾йHöø󷧚«&\$\\𖛤[µN±6¥6>¼·Hù!¬ýKNߐ3臜r䟥>7证㺤õ:&䚤%7k®(װMÁ£˚[trp.I¿Qʿн¨YÀ(Ꟑ榵#ܤ)ϟ)mºª,ºሲ?dlH𿨪r]ҔX`ˈ!@B휤\\|»§ûKe茘\0ɜ0I¸놰h¦Lkƈxh&ӱ˛91\0p\0ā¸Ur÷@8\0j\0Ё¾9{À\07!\0fĽǠ\r@\0006Ԝ0t¿²ú¥򀵗˯򟀃򺍿3۰IRý岌	/¶`½ɽ`弋÷6\0ȁ̀À ¥÷˱\0q/°´Áټ\rÀ\0000ӰJ^ø 겦L ÷0¢_\0i򼥮̂\0g/柤aS̜nøT􁜃9³&̴À\0܁́I~򺜰̬ɯP&!L5À\0¢Á)³\r廂L؁©~%¼°p󁦙̟ñFc4ĩ򹦑KޘԜ0Ң쇩󑦏\0000\0b\0ڣÁǲû&*̓0 ً̃3\00050¢aP	2üfL&\$1h¼¿IҺÀ\00050~`4Ǚ{󜲦QK옯/ޣ􆹋S&̆װ¦_䂀`\r <\0c¢\0¡\$Ǔ퓭慌|0e¤	¦_Kޘý0~`ԃ)3fV̽\0b\0棌ŹS&K̲Ӳf_¼©|2ÀLu·1`dÙ󰦌L<篲_lƙ~󚀛LÁ䄖cü¿ɐ`\rfhb\0jaș毜00043_Ą3\r@4=0jbLÀ§³5fJ\0004ѳ6^鯹Ӏ槌&90Za¼ɹù¦X5ۯި(is%@LŘ/ºg\\Àis\rf\$M(µ4Nd\\ɉ\r¬&p̑ƠđI³\"fKޘU0hĿùD拜0004q0\"bҹ}s	&\0002\0i/zaüɩӥ&\nL嘏3^jtӓaS&̙5*cT¿顓\r@˵4\0Ҧ􉐆SV¦5̾\0ڪTʉ®:檌\0g4jfҙþ&´L?5¦a,Љ3!湌3fa쑠3?¦K󚝰_D͔γf§ˡòkܿI¦3A%úK0¬ĕ9|	fLµy0:j¼͹©󺦏L\0c3ҬlЩ³g渍 鵎c´׹³.漌ײfg¾Y«S\"f{̜0«3htƀ7¦y͢�jTѩ󖦲Ll(«k	¢\0¦\$ͱӵöfթ¤%÷ͼ5¹X\\˩sS&¤M4öb¿飳rf͆A1\"iԐ¹³[f0L<³¸ى±sX&0·/櫔ɉ󾦌͋10²û¦͋١2J`Dҙ󁊞Μ0¯1iԿ3+f󍥙뱂n\\߹SfM¯+4¾j|ݩ湌5jbd≽Sfn̓5ÿ0弴ᙃ󟦑%W6þi´¬3v朌þ8Jh¤ѩsGH㎜00Biԏ陓P秌(ӱ6müܕbͅLqͰj쑹·@f5MQ±2Bdᙧs¦tLHq5¶l<㉨󐦸K䙧0nr´Ài@&ͨZkþs\\Ɣ󓃧EΊ3ֲ¼ƩȒü熌r5®p􆩥S\0%úΦw3¶cl㹄ӄ&kL¡=1a¼穄³t潎M3`ѩgΓ»9\ndÀyѓA¦λ0pLؙ(¦ǎQM7öp˹Ӭ眰Ί2\"kÀù»3:&´N:o4zv´̹37§4Mq%:Nhtۃ_󜮦®́¹2Js¼ÁI&썙ﵜnl,̹γ'BM;÷8Nw|繓3¯&L񲦳\\¿9-f̫0¡얉ù楎mW/nt݉ɳ¼f¨N3¾l\\ʩs¦'/6°/rÁyܓ	§.ο-7otʩӏg'̸;5Xü膩»S§5Β밖eDӉóvg\n0Z`ܒ9gMº	0ªj4䩍s*¦=N\rs2vwԪ¹ѓN¦ϥ=;nj싹ų5¦2ͷ7udЙ­撏L¯=\"o¼�s&D=÷/⣴ӹ曋纳3fd��Ӽ描\n»7{TͩӅ¦IM#Ѷ`ÀYƳ+§¸O#9ȩ숉£f&ꍞ\0k¼登󼥳MM%5jx,ڢ8Ӂ恏d±<d÷©¨ӈ峏½<z|	'³M9®m񙦳e&?O}3꧴󉘓&®ͳõ/o¤󙞳\$§̞վꤔΉ²k楍¾+>ReqSٻ ¦\$NټFpüәĲù描X`x,ú¹鳵']͜0Ѳ6k¤ҩғ¦񏋾պpõfNƚ﷦iĄ閲ù服Ųbf쐉¬3H樌񘏯²t¤񙼳欌̚:h\$¾Y¬=泎\nC=>qüÿƖ®&0«ˌa͉ȓõ§VС]3ΫڙSR(Ξ2.n¼¿:sx瀍ᜱ@>lt噸sئ̀ٶ䩉¨\"%öL՛¹3vaUЄ��LHG4pt퉼㦺MI@®xԷYs§rϩױBn4¾ɕg}Mٙ½4.}֙¦󑦰Kø=6z\$ɉ򳅦ЇGkə󶦸Mt 󰖁󙉳[&\rϸ!0>mߩf-M\"2®h��Sÿ&͎+9N{m9¼2ú&䏊¿>ܼ鲓秒Ob¡E<ƪļӘ§fNg<h4ωԳ¨潌С7=s콩3樊̒2܆ɱ®¦9L'?r¤ǩ©󶀒̴@vg|əü³+蹍sÁfzlΉº&LjÁ::��©h\n@£F¢ ^Tk¶ꮾ]ÁՑEPþKµٜ©e¨¾N#\nƝ@Y30®cnM٢��è2)hf¹°=F¬I񞬅þőXP?iɄ¢8ʎ\rK蝍xY3š¥´(I쑨3na莑ؔ7,B)+¢NǞª&°٢BQ´ ¨ȸ͞Áµ³\r^¸ɞN#¶ORjᭈ\n´тø( ā¯jøɚ\nö¦wԖƑaDi핖8Ԁ,À²'< +A3K㨲񟠁KAµ%l٨¯L½ªN;ú'(ø(Ѷ^Βµ¬%F+Ū��#(ćw`)~U񬈨º¤K9ªü´da¯y_͆ukq2÷لI£*M1«0µF수I執	ҧx0𣚽3_ùFĭL\r¸+Hѱ\0°Ӊ BõJpmÁߏ䍔ŕTr!Áܰ±G&E曠֠򇻑m!¿а¬õ¨ꓒq°,P蓔NH­/ɄQ»^x6̅¨򂙃\n»Ԛ%V&¬Xp5ۆ´ϔУv\$\\\0P&C] À#2»¥Gü©4\n9K¥µ;jú:@ԼYS`O񼤦Pk@إ2Á¤8Ȁ\\§4|^Q#e&Ü$¡2­0״¤>ʉ¦3.· x3\0P׭§JJ¶Hʩv¥¨ 65lZ4¶)«C0&Y.!½µIBe9KFEÿ͙(¯Q'IM -´YċQq¤ѱ󰂋}D¨·\0 ¢ݿ󜮆º⥙]c»¡K3%ݔi/Rq`F+\n12̹臒u¤܎xL¥ x?dݩ8±d'&ÒFнiCÀФ̋½QX ²h¤ɢ\\줩Mоdüɔ?¢t<p.Lµ;jɖ¼ª5#dɭ­쯦6H5¦E¬(D)¤¶Fʐ󌈻  0¥cA­d(ΩFµ,VR,ø:²7\n¥5\nw\0¥ªϝ}(£l¸ٖVEDʆcö£²gWƨqTpЕs(+Ph ѕ¥ց@¬¦¾̾Kɦ (jퟨ�º貑f¢߉úN䗆5!ú@´^пfµv¿􉩷AV^²ԅ¬¥鴀hӉcК!> 󑰞柨?ZbЫiuAN¦Hƭ 􎆗@脒D_hÞ8fw詖2¢ÁLx9໔ŗþQY¦^Ʋ%\"B²¬送\0V»Mv«V´Ř!Ҍc` U3ö 췉ªÀ䠹M2¢øր¡úڲSXbMʭ©9˺XZ/¦,½\n�O{)+Rû¨a¨CH2tŚöµყH¶Sƾ ¾ý¦KkHb̫¤瓒e𖺚5ZOlhW˓Ʀ,ŵ}u`Áx¥뎘M0ذ^ilÁZLZ«ösԊiд;aNڸõO􋩆S`N)}ݸfԲ?Ӟ¤·Kü\nµ3x0Y	¸2ի­N1f+쩡Ss§֎´갔¤`ö1gǌY7пжׅf¦a8¨DЯ §Ұ'Kɯ*{lUý¥õOE;z-J\n#ö}t©÷AµhzOJc(4¬ܩS裯ٶ­1Z~´÷񓘆訶奚cөৼ\0®%@úf0SÀ*ө§TҮ75ߘ¦kL\re3sõ鞲t¨H½b,e´oj²)¦½fU=x6´өªӏd^Ҡ}CJkL´¤蔭b£MfS\"kµ©©ӟ¨5֚½DF4¨ݩ´Á¥¤ûO²D冐[؈SƟ֛͆«):n􁠒ԙ§¥OꢃGö!ԧ ғ¨ÁOqìLͩű©§U92䘻Ӕ¨O£¦ꎰ^ªӟ¨N=ARuiӓ(§R杝BºT÷)ؔg¢)C啅H꘬쩤Կ§Ր(]¨9@iޑe¨O¢U<:ըj+A¥§Q*UEz{󚔓΢R©M=ÿ,\\)ꂄ¦ݯD̹©lԸ`YO£t!ztT¿h·S멳Oµv¼!gZ4S.©¾򚾍)úTτ9D\"͌(C\0(Á/G^bý@jcUi1÷¦AH¥Mf+ߪmT낒ÞE2ª4멼A¨9LȮ\r3J=TF ԥ&S¶B5=jp/d¨Pؾ탊ퟚ�SV¨{SªøºiÁþj!¿ªT¢5 x 5jTKbT^¢}Lz@p@i¸TT]בZ\rEꈴ¦ªӲ¦÷G©な¥5©ÁU-aQ¨½Fq5*:0\nh=Q¨퇦z칗z°¤&̜"		8r,9׏07ªsN^ª\rGª¦Դ¸՝©R¬ª𾪁T򨱏¨ՐZµEꆒt¨¡U¤½NZAjPU©EU\"¨-Jz£µ*`Ԭb®þeU:[Á=ꪔµ§¬½¦¨}±\0鼃aOzsR2i»ꄲf^z̊¬`\0윜׷ú\0&¬­6|´¾Z7/t漂¬­R²̭þ¯1¥úVZ¬񷪤k»*\0TFܻ¼°va­;א´բ®-I;*)²kº¯jg´ӝ{Q¨姈\0ߕÁ|H²ú\n¹4)³/L«oV抳	\0ྑxºςS&:DժI Rû`n|¹A䦴^jU¢�$u]ʴyٳS\0¾V6­)½Bp`ժ«¼½d;V7վj¶³q¦ªʚMm꾔n鈳y¢񘖇µͳ̓\0¾¼sM'욘Jf0q=´e³'fgXqŗ)Bi)t2_䍽¡y=0	*jUȧ܍þ¯µbzõ}ø֯d^ÁN¯cfu|Iɴ(QXʍ-Ǎv+ԑ\0Xnudz¼˹!\0CFϏ¸ŏ¢}jєȦPx¡>ʾ��\0Y©5YFÁ>²ҿ+*\0M¬¿Yb³8鿕먖hh¨⡚)Ք ~¬ Mְ꒕H3¼`%Lγ̴F«@/\n¬µJݏе¡+EV¬ހ²³0¥ÿrI¸ՠ_ݚM#9lkLVg'K構:Ҵ؇ѱ­(ϲEMZԌÿ)tѱ¨PÆ󳂬lk\$V³&oV±­kf\0õخ°୫ZõµY6-Tu °ଣGm\rlr¬lY3֨T´ᖰü¹:QeK2«mPڡUB@#֊ªR6¶U`췙·地¼öü¬o©ªѥ'=[¡Q±?¦ġ+²­ݖ°«Ӭ\nޤ䫻¨��½µ8ꠌøɣVµk⌑+3º>ₙõܰW¸el*Ȁ\nøH'؟}û Ƣu«3yf'Y.§C1قę]ҁ¬\0¹+1Kݩ/«G[)峆µº٠Э5Z­¥^兠³5®YHU¡«\n8Aük2»\0§[򐭡ª鬒ퟥ�6ö>}qý­jіKh@ȡS»#¡EQ¼§Q2¯KµU֙b³¼&Vý#p+朰³k Lö®¢öuڍ6+Á¦ʩz˟֠MD{//®8©Y8\n¤⪸т¥³)C(Uω³^ܘõx²lխkד¬^V¼l⌄«̜0T­^v±¥vMh£ߨ瞱£%\"*K\0õ\0We^Sº**L+¹w´ԫo^n½`ûµx+°5þi_[)£ɦ^«,ªηֽf+Ƕ©Ӊxa=|凪@VK\0 \\\$­u묖d꾞£UdµC²%Rn&²Lý#\$'ú«=RqbK2÷iFuüI®ʀ߆yӎªN5.¨ʒc<ÀI¥'R}4§	ö«&Á!«0õ´=Y¯@«eI¹}v󕇫͖K􇎵Ř*½l )ӖʠVÀ¶¯y3úSu¼밁°6N«PV⍁AX¸´ϛ{ÍG0?¦Ӟ¨|&7XD´¸f;`yõ*𰵙*櫝]!\0Bi*؋­d͹¨ZǬÀ*ҩ_aU®Bgõ­Xd؛°,֍¡­K\0싲䏶т\0褢0?°ԎI%\rՙǘ°՛ø>ý֜nöX!©Ӗ²4kΙ휰G	ꁾb𦡔³W&aÁՇ&I𵪉³2`xM6:֫ȕ?׸dN¶RøSzµb顲¥Ζ-*Xv뎘`¬¦µe\rÀ¹\$btZYö,À<صȁÁmû \n½SEf¬¶֠aZ¯+jJaʋ÷5SºdhK±@n\\)he_٪Tȕʸ01ºI¼*¨q pঀ΁±Ԇ80@3ʡGؔ̈DЪQt;aI*꿆 ퟡ�氀6%aڸܤb2I	¸�>Ϙü¯öȘn[\"³lْVÿdP,[# }ºY J\0ډtۜ$Ӯ,٥²4Œۣ!\09²ed~ɼǠMם;:31úɠ!²³Yl 8²0ʭK'ܲ,¤ٚ²d&µ맶Q씙Q²IdΊ8{)¶Oٱ²­dªʰ[&֗엙\\Ienʚy뭇FfYke¢c\r,V,¸¯²礠1ܔk.BЬ©YB²õeڋ啻/\0콡��N˕˰ö_¨j٨²õeV̜r/3¬Ǚ]³+8̭{1vQ¦°³7dRh]dv`¬©а²Ѵ.̕, 慙P³Uebh]4vd¬֙³?djh]뵶`慙c³wfɕy֧¬ߙB*{e¶͸c¹À¶q,ٙ«fȅ9À¶a¬♆f¦έÀ¶k¬왒²Oe¢p-«9 炙º³»eNp-뼶P¦ٚ³«dRϭ˺6z'\"Yಟgϭ>v{&Tٶÿe¢_\r»=vQ­\0ِ²ûhȵ;@6~­ٔ姊Э©ޖ¬ҙB姒Н ܤ¶-\nن姲ϝ¡뻶x繙ⳟhJ̵¢K,VZ&aٮ´3h¢˕¢«9Ýڭ´1hДػEmY²´qh5¢ûF³1흌ϴvtfҽ¡C¶¦ZG´ohFҜԫHö-(\0_ɨúҬ΋H6�洸~tE¤«6¢-\nZc´hғ¡»-¢-*ZK²5:\"҅¦ǳ¢--ڕizӵ¢»<f¿헚_´ùi͝§ˌ¶ lûڨ´ùhrӴڻMs­8ډ´멊ӌ½둶­=͋²wjT¨+`­Z_µ9jԘ[)x펚³WjrӭKS£mSڢµ°ʥkV¨쀚³´9jΓm«;N6¬헫÷´Sj֕õVV¯­>­Cµi*Յ£;4v²m''µjޖ%«X®�½XCi֝­{R°-kڈµ²°²n]¬W֓mok6Յ¥+\\¦'٣µӫR֭­k\\mwZ㳔\0LӉt1-gZöýk⩄½ù;­XLaµ뫞k½°{\\\r¬-Z昜rl6פ˻bɦ1[w;ذ9ƧQۜ#lrj5²\\Vµ洛 µѪúhu²ZVɭxZڋlӅ¯ۤzZ¾*{lr_¥³döµ峛8¶Wkhõ³^V¹¦G[8¶ikּcH­¥ڔµa1ڝ­i6ϭsMp¶mפ¿ivѭZ׃lra]¶i֯¦5۠¶¥mzڥ®Ʉح¬[˭ڥ«	vǥ¶ǫ^_e·Klö¹¥󛴶[m򛭱yöݭ¸ھ5ÿlrbm¸nö¸&\$ۈ¶÷k²_¸oö䭀ۚn\"ܪ°Vƀ[#XYd.ܥ«KYjýÀ[&NۍºyV譊\$¶b°	%¹sö±m®O=nzݵ¬«f¶槑ۜ´uk®ݽ¨«x\0nӏ[·n¶ԕ¼9{¶¶푛µ½krܽ©s¹-蛶ׯNݵ¼I¶��ڷ[kҞTӋvV�[ᶓoܝ¾Kgv等[淓o_m¹ɂü­񙼛j^ޭ¿i©ö筒ۡѯ򞌽٠ֿ-·[0*ߥ¿[töͭퟳ�9l栭¾»v֏nۯ»m୤`}Aڙ᭜"ܼ¿럷\0朜㮂ᐬ[nÀ3ZSp¢ݽºI筥[ᶙpڟM¿kj\r®\0̿¸-1þଇûu¶֮\\\n·m~ߜr¶7mӛe¸pöd͹˭wm֛o¸qa­ċxޭퟶ�9mö⭄V筃ۡ·q┑Vý¦ܤ¸8J㥃sS4-ٿ¸뷮㽆²d熮\0¸,tfੑr§Fn9,,:3qbԍ¿y·!­۹¸÷nꡁ¡{םmߜwjf㽟ûyV¦nAۉ¹,¦ޥ»»ֳ·¥rnޥʓ?.=\\µղ~`凹~·f\\´¹r٥寖5\$ȕg{Ցڂ5^ª3hTÁܮ`ذU¨·T^¨呶¶'*U\0Oª­RBqL\"į5b·UҨ+Cú顷Z4/wcª¼½QxE®b4_sH¸-õE뒑ªfQQ\rS©U頭°l¹Q⪍(揂¥9®ժȁeлZ_Wi%Rö«d뚰*·՗¹©T^«®eՕG¿L²񖽔헷՝´˪¯jM·JJ S¹~Ƃ鵮ʜ\´᩾07a»L§e8jk·iIA_¹M.铉¢lͯ鑓йU曍RºRX®UͺÁr轸k䟩³°��ºΫ9no±*¦-ujMՋ챩¨݌¹֌\"\nՎ꒷S.xܳ§\ruµ;)PV.Pºou¾©¥j[¯>)΁v¤剢õԋ 宁/׺t®󜺶WB+ÁS±º¾B뽘ú᪏SÀ·)옊xTk®¦]Q»;M	½k®ѫI¿º½Y@u¥Dkݚ\\SºGRQý?¶&T=꥕þºgR§s\r*ηn ]ºaϴ򠕓úlZ䦳u¢U̫¨��\\ϨTr\rō:°mn©ՕuuZOeD«¬9.o]c¨wn歎f%7:כKu }֢µ¨҂땛ʜן.÷S¤¡5G[0p鄒^;v¬«±ª>ݓ§}R~Nܝ¢U'הÿR~탚´U깁ۻN\0¢	6(j(ꂜ$®-J&杬¶¿¤ဨ¦5t®euû¿̞鬒8e쏶鵼tk񮓩¥J6õ))QW~lB¥7G\n¼ ֽ,l@亜0ƒS¥­ ¼n£+K\"賄§Hᣅ´nLѮċ¡s[¥yr7[A֤]mK+ɻۥﴑ\"²%{E1wK`Ӂ巰r|!µ;V@µ°;bN-:bݦƷ!ނ|̈l뻻Ω¯C6eNذsڧ(␛¸��!DU鈉W¦f^4	ö��<w¦\n#Hxح»õ鈼ףᴃ\rÁ绕𾨂Ĩu=币 £·¶õúA;ҩʄ^[xr끙¼pöqÀB褽Oþ¹A0Ā`1]DZݑ_¼¶㏷uݒ6؇CÀaJ®rz÷(£ל\Ɂˍ⏘ª|̈պ·¬þR½ƞmѽ嶡@\0=5l!zQы|럎	ɜBúWر��݉Ҁʂh촧ݥ (·ĜrGEū§|@U�71ǡ;R9QC¸47Y曺_#½칍֠\"k巈;au©ݤ±³^¸j끃¸sǙ❦6artْöhЏضǰy­{Z啁唦¿t؅fù¯蘮Y=«¾¨	)ºۦȠ\r�7yRúУ»뗀_[W,	r󝶗@nˇ>_a#|yӸ&䈗ۡ͞_DIױ§ ?Λ\0º*u6Ж܈߀뺺V´c:­YL\rjչ÷.§¯dh\nBû¼´Uᮯɟ½bYüµ󫴡񜘃}¿G¡¬찧¸wүUſ,U}nK ttGȿ!~Ág@Ԓ/ٟ±\\40´KěõüoڄｙLõ󫶠 ^º⯝33+7¨ΛǮz즴 ¡Ўʛ%ߒ¿¶\"þ蘛蠳/rp,վe񒌤R᱃¿Ͽ°g­þõ B½­8*8Gɦ¡ރۇ¦±rÿ𸻼饐K_玼䢔\0۷ǯ/ʊ^l] ©©\$¾yù𙱘Mϯ7֒wz��ݍ³[LޣGzV豷la L@¾H	\\HPѸdC·üᜤcc±Ɖஸhfɜ#9zɲ4]5\0Ó\0¬ŊÀidiøQ§Oؑ0`Q¤¡ªþüÁ.8Áy򲙧Wªg¸3ÀZͿ孌®\r᜶\"Á¸kiZA.+£f<O7Dئ¾À6O1ǘ`HoЃŀ慮��¨§ߨÿ`Sڪ`(«¢𽞵Á\rv¬>VJ«\0n½OUºÀ(`λjʗ\\h»G®¯JB򢫊=�ݰQ߿{ AJ¾ÁQ/~²T&j𜞮EϼzÀC綸/°Q߶݉ùwᰛ߁@a\r Gd6ň闥\0000®­σޘ\$⸸ی`㷫ݨ!Kù׽½[ª\"g ѧP=wrퟡ�õd؀@¢ٜ0GIQNÀ7.,ba~~lK!	»ퟠ�)MzȕJBΘFAùXƂZOA!±²؂ pxi´ÁøFÀ¢iýg􌻚JÀcT/»+r핓țFs\$	¨&ö=ը¡Ė¦XXIޫ¥ ኃȕ[pø6T-q4Ђ;ąPU\0QÜ0ĂЇ\"	½b{g ) \niah4܌1l+л@9ᩜ0NUݡȄNk\r`OKJúg֜.ˣ忿h²jÉ»\rR®_t,5 ֥Wobµ>U°悉w;5´բs©a¨厔§");}elseif($_GET["file"]=="logo.png"){header("Content-Type: image/png");echo"PNG\r\n\n\0\0\0\rIHDR\0\0\09\0\0\09\0\0\0~6¶\0\0\0000PLTE\0\0\0­+NvYts£®¾´¾̈ҚüüsuüIJ÷Ӕü/.üü¯±úüúC¥ל0\0\0tRNS\0@昦\0\0\0	pHYs\0\0\0\0\0\0\0´IDAT8Ք͎ǻEᬬ϶õ¤p6G.\$=£¥Ǿቌw5r}z7²>P壜$³K¡j«7üݶ¿̎̿?4mѷt&3!00^½Af0ޜ"彑�𪠧4¼⯥E賈藘(*Yӳ¼¸	6	OW¢Ɏ܊m¬r0þ/ ጨ\rXj#֕mʁújÀC]G¦m朰¶}ދ¬ߑu¼A9ÀX£\nԘ8¼V±YīǄ#¨iqޮKQ8J౑6²晰§`P³bQ\\h~>󺰓ɀ£¦¼¢سGEõQ={*3벣7÷\neʔL肊~ЯR(\$°)ʧ ÁHQni6J¶	<׭.wǉªjꖭ«꼭¿?Sވ vÌû񆩧ݜ0֞ձ«¶)ª۝÷U¹92Ѭ;ÿǍøµ£!X˃䚜ÿL񄮻tæý/wÓ䬗R÷	w­dӖr2ª4[=½E5÷S+񗣜0\0\0\0IEND®B`";}exit;}if($_GET["script"]=="version"){$o=get_temp_dir()."/adminer.version";@unlink($o);$q=file_open_lock($o);if($q)file_write_unlock($q,serialize(array("signature"=>$_POST["signature"],"version"=>$_POST["version"])));exit;}if(!$_SERVER["REQUEST_URI"])$_SERVER["REQUEST_URI"]=$_SERVER["ORIG_PATH_INFO"];if(!strpos($_SERVER["REQUEST_URI"],'?')&&$_SERVER["QUERY_STRING"]!="")$_SERVER["REQUEST_URI"].="?$_SERVER[QUERY_STRING]";if($_SERVER["HTTP_X_FORWARDED_PREFIX"])$_SERVER["REQUEST_URI"]=$_SERVER["HTTP_X_FORWARDED_PREFIX"].$_SERVER["REQUEST_URI"];define('Adminer\HTTPS',($_SERVER["HTTPS"]&&strcasecmp($_SERVER["HTTPS"],"off"))||ini_bool("session.cookie_secure"));@ini_set("session.use_trans_sid",'0');if(!defined("SID")){session_cache_limiter("");session_name("adminer_sid");session_set_cookie_params(0,preg_replace('~\?.*~','',$_SERVER["REQUEST_URI"]),"",HTTPS,true);session_start();}remove_slashes(array(&$_GET,&$_POST,&$_COOKIE),$Xc);if(function_exists("get_magic_quotes_runtime")&&get_magic_quotes_runtime())set_magic_quotes_runtime(false);@set_time_limit(0);@ini_set("precision",'15');function
lang($u,$uf=null){$ua=func_get_args();$ua[0]=$u;return
call_user_func_array('Adminer\lang_format',$ua);}function
lang_format($Ii,$uf=null){if(is_array($Ii)){$vg=($uf==1?0:1);$Ii=$Ii[$vg];}$Ii=str_replace("'",'’',$Ii);$ua=func_get_args();array_shift($ua);$jd=str_replace("%d","%s",$Ii);if($jd!=$Ii)$ua[0]=format_number($uf);return
vsprintf($jd,$ua);}define('Adminer\LANG','en');abstract
class
SqlDb{static$fe;var$extension;var$flavor='';var$server_info;var$affected_rows=0;var$info='';var$errno=0;var$error='';protected$multi;abstract
function
attach($N,$V,$F);abstract
function
quote($Q);abstract
function
select_db($Lb);abstract
function
query($H,$Ti=false);function
multi_query($H){return$this->multi=$this->query($H);}function
store_result(){return$this->multi;}function
next_result(){return
false;}}if(extension_loaded('pdo')){abstract
class
PdoDb
extends
SqlDb{protected$pdo;function
dsn($kc,$V,$F,array$Lf=array()){$Lf[\PDO::ATTR_ERRMODE]=\PDO::ERRMODE_SILENT;$Lf[\PDO::ATTR_STATEMENT_CLASS]=array('Adminer\PdoResult');try{$this->pdo=new
\PDO($kc,$V,$F,$Lf);}catch(\Exception$Fc){return$Fc->getMessage();}$this->server_info=@$this->pdo->getAttribute(\PDO::ATTR_SERVER_VERSION);return'';}function
quote($Q){return$this->pdo->quote($Q);}function
query($H,$Ti=false){$I=$this->pdo->query($H);$this->error="";if(!$I){list(,$this->errno,$this->error)=$this->pdo->errorInfo();if(!$this->error)$this->error='Unknown error.';return
false;}$this->store_result($I);return$I;}function
store_result($I=null){if(!$I){$I=$this->multi;if(!$I)return
false;}if($I->columnCount()){$I->num_rows=$I->rowCount();return$I;}$this->affected_rows=$I->rowCount();return
true;}function
next_result(){$I=$this->multi;if(!is_object($I))return
false;$I->_offset=0;return@$I->nextRowset();}}class
PdoResult
extends
\PDOStatement{var$_offset=0,$num_rows;function
fetch_assoc(){return$this->fetch_array(\PDO::FETCH_ASSOC);}function
fetch_row(){return$this->fetch_array(\PDO::FETCH_NUM);}private
function
fetch_array($ff){$J=$this->fetch($ff);return($J?array_map(array($this,'unresource'),$J):$J);}private
function
unresource($X){return(is_resource($X)?stream_get_contents($X):$X);}function
fetch_field(){$K=(object)$this->getColumnMeta($this->_offset++);$U=$K->pdo_type;$K->type=($U==\PDO::PARAM_INT?0:15);$K->charsetnr=($U==\PDO::PARAM_LOB||(isset($K->flags)&&in_array("blob",(array)$K->flags))?63:0);return$K;}function
seek($D){for($s=0;$s<$D;$s++)$this->fetch();}}}function
add_driver($t,$C){SqlDriver::$ec[$t]=$C;}function
get_driver($t){return
SqlDriver::$ec[$t];}abstract
class
SqlDriver{static$fe;static$ec=array();static$Nc=array();static$pe;protected$conn;protected$types=array();var$insertFunctions=array();var$editFunctions=array();var$unsigned=array();var$operators=array();var$functions=array();var$grouping=array();var$onActions="RESTRICT|NO ACTION|CASCADE|SET NULL|SET DEFAULT";var$inout="IN|OUT|INOUT";var$enumLength="'(?:''|[^'\\\\]|\\\\.)*'";var$generated=array();static
function
connect($N,$V,$F){$f=new
Db;return($f->attach($N,$V,$F)?:$f);}function
__construct(Db$f){$this->conn=$f;}function
types(){return
call_user_func_array('array_merge',array_values($this->types));}function
structuredTypes(){return
array_map('array_keys',$this->types);}function
enumLength(array$m){}function
unconvertFunction(array$m){}function
select($R,array$M,array$Z,array$td,array$Nf=array(),$z=1,$E=0,$Cg=false){$ke=(count($td)<count($M));$H=adminer()->selectQueryBuild($M,$Z,$td,$Nf,$z,$E);if(!$H)$H="SELECT".limit(($_GET["page"]!="last"&&$z&&$td&&$ke&&JUSH=="sql"?"SQL_CALC_FOUND_ROWS ":"").implode(", ",$M)."\nFROM ".table($R),($Z?"\nWHERE ".implode(" AND ",$Z):"").($td&&$ke?"\nGROUP BY ".implode(", ",$td):"").($Nf?"\nORDER BY ".implode(", ",$Nf):""),$z,($E?$z*$E:0),"\n");$Th=microtime(true);$J=$this->conn->query($H);if($Cg)echo
adminer()->selectQuery($H,$Th,!$J);return$J;}function
delete($R,$Lg,$z=0){$H="FROM ".table($R);return
queries("DELETE".($z?limit1($R,$H,$Lg):" $H$Lg"));}function
update($R,array$O,$Lg,$z=0,$xh="\n"){$mj=array();foreach($O
as$x=>$X)$mj[]="$x = $X";$H=table($R)." SET$xh".implode(",$xh",$mj);return
queries("UPDATE".($z?limit1($R,$H,$Lg,$xh):" $H$Lg"));}function
insert($R,array$O){return
queries("INSERT INTO ".table($R).($O?" (".implode(", ",array_keys($O)).")\nVALUES (".implode(", ",$O).")":" DEFAULT VALUES").$this->insertReturning($R));}function
insertReturning($R){return"";}function
insertUpdate($R,array$L,array$G){return
false;}function
begin(){return
queries("BEGIN");}function
commit(){return
queries("COMMIT");}function
rollback(){return
queries("ROLLBACK");}function
slowQuery($H,$wi){}function
convertSearch($u,array$X,array$m){return$u;}function
convertOperator($Hf){return$Hf;}function
value($X,array$m){return(method_exists($this->conn,'value')?$this->conn->value($X,$m):$X);}function
quoteBinary($kh){return
q($kh);}function
warnings(){}function
tableHelp($C,$ne=false){}function
hasCStyleEscapes(){return
false;}function
engines(){return
array();}function
supportsIndex(array$S){return!is_view($S);}function
checkConstraints($R){return
get_key_vals("SELECT c.CONSTRAINT_NAME, CHECK_CLAUSE
FROM INFORMATION_SCHEMA.CHECK_CONSTRAINTS c
JOIN INFORMATION_SCHEMA.TABLE_CONSTRAINTS t ON c.CONSTRAINT_SCHEMA = t.CONSTRAINT_SCHEMA AND c.CONSTRAINT_NAME = t.CONSTRAINT_NAME
WHERE c.CONSTRAINT_SCHEMA = ".q($_GET["ns"]!=""?$_GET["ns"]:DB)."
AND t.TABLE_NAME = ".q($R)."
AND CHECK_CLAUSE NOT LIKE '% IS NOT NULL'",$this->conn);}function
allFields(){$J=array();foreach(get_rows("SELECT TABLE_NAME AS tab, COLUMN_NAME AS field, IS_NULLABLE AS nullable, DATA_TYPE AS type, CHARACTER_MAXIMUM_LENGTH AS length".(JUSH=='sql'?", COLUMN_KEY = 'PRI' AS `primary`":"")."
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = ".q($_GET["ns"]!=""?$_GET["ns"]:DB)."
ORDER BY TABLE_NAME, ORDINAL_POSITION",$this->conn)as$K){$K["null"]=($K["nullable"]=="YES");$J[$K["tab"]][]=$K;}return$J;}}add_driver("sqlite","SQLite");if(isset($_GET["sqlite"])){define('Adminer\DRIVER',"sqlite");if(class_exists("SQLite3")&&$_GET["ext"]!="pdo"){abstract
class
SqliteDb
extends
SqlDb{var$extension="SQLite3";private$link;function
attach($o,$V,$F){$this->link=new
\SQLite3($o);$pj=$this->link->version();$this->server_info=$pj["versionString"];return'';}function
query($H,$Ti=false){$I=@$this->link->query($H);$this->error="";if(!$I){$this->errno=$this->link->lastErrorCode();$this->error=$this->link->lastErrorMsg();return
false;}elseif($I->numColumns())return
new
Result($I);$this->affected_rows=$this->link->changes();return
true;}function
quote($Q){return(is_utf8($Q)?"'".$this->link->escapeString($Q)."'":"x'".first(unpack('H*',$Q))."'");}}class
Result{var$num_rows;private$result,$offset=0;function
__construct($I){$this->result=$I;}function
fetch_assoc(){return$this->result->fetchArray(SQLITE3_ASSOC);}function
fetch_row(){return$this->result->fetchArray(SQLITE3_NUM);}function
fetch_field(){$d=$this->offset++;$U=$this->result->columnType($d);return(object)array("name"=>$this->result->columnName($d),"type"=>($U==SQLITE3_TEXT?15:0),"charsetnr"=>($U==SQLITE3_BLOB?63:0),);}function
__destruct(){$this->result->finalize();}}}elseif(extension_loaded("pdo_sqlite")){abstract
class
SqliteDb
extends
PdoDb{var$extension="PDO_SQLite";function
attach($o,$V,$F){$this->dsn(DRIVER.":$o","","");$this->query("PRAGMA foreign_keys = 1");$this->query("PRAGMA busy_timeout = 500");return'';}}}if(class_exists('Adminer\SqliteDb')){class
Db
extends
SqliteDb{function
attach($o,$V,$F){parent::attach($o,$V,$F);$this->query("PRAGMA foreign_keys = 1");$this->query("PRAGMA busy_timeout = 500");return'';}function
select_db($o){if(is_readable($o)&&$this->query("ATTACH ".$this->quote(preg_match("~(^[/\\\\]|:)~",$o)?$o:dirname($_SERVER["SCRIPT_FILENAME"])."/$o")." AS a"))return!self::attach($o,'','');return
false;}}}class
Driver
extends
SqlDriver{static$Nc=array("SQLite3","PDO_SQLite");static$pe="sqlite";protected$types=array(array("integer"=>0,"real"=>0,"numeric"=>0,"text"=>0,"blob"=>0));var$insertFunctions=array();var$editFunctions=array("integer|real|numeric"=>"+/-","text"=>"||",);var$operators=array("=","<",">","<=",">=","!=","LIKE","LIKE %%","IN","IS NULL","NOT LIKE","NOT IN","IS NOT NULL","SQL");var$functions=array("hex","length","lower","round","unixepoch","upper");var$grouping=array("avg","count","count distinct","group_concat","max","min","sum");static
function
connect($N,$V,$F){if($F!="")return'Database does not support password.';return
parent::connect(":memory:","","");}function
__construct(Db$f){parent::__construct($f);if(min_version(3.31,0,$f))$this->generated=array("STORED","VIRTUAL");}function
structuredTypes(){return
array_keys($this->types[0]);}function
insertUpdate($R,array$L,array$G){$mj=array();foreach($L
as$O)$mj[]="(".implode(", ",$O).")";return
queries("REPLACE INTO ".table($R)." (".implode(", ",array_keys(reset($L))).") VALUES\n".implode(",\n",$mj));}function
tableHelp($C,$ne=false){if($C=="sqlite_sequence")return"fileformat2.html#seqtab";if($C=="sqlite_master")return"fileformat2.html#$C";}function
checkConstraints($R){preg_match_all('~ CHECK *(\( *(((?>[^()]*[^() ])|(?1))*) *\))~',get_val("SELECT sql FROM sqlite_master WHERE type = 'table' AND name = ".q($R),0,$this->conn),$Ne);return
array_combine($Ne[2],$Ne[2]);}function
allFields(){$J=array();foreach(tables_list()as$R=>$U){foreach(fields($R)as$m)$J[$R][]=$m;}return$J;}}function
idf_escape($u){return'"'.str_replace('"','""',$u).'"';}function
table($u){return
idf_escape($u);}function
get_databases($ed){return
array();}function
limit($H,$Z,$z,$D=0,$xh=" "){return" $H$Z".($z?$xh."LIMIT $z".($D?" OFFSET $D":""):"");}function
limit1($R,$H,$Z,$xh="\n"){return(preg_match('~^INTO~',$H)||get_val("SELECT sqlite_compileoption_used('ENABLE_UPDATE_DELETE_LIMIT')")?limit($H,$Z,1,0,$xh):" $H WHERE rowid = (SELECT rowid FROM ".table($R).$Z.$xh."LIMIT 1)");}function
db_collation($j,$hb){return
get_val("PRAGMA encoding");}function
logged_user(){return
get_current_user();}function
tables_list(){return
get_key_vals("SELECT name, type FROM sqlite_master WHERE type IN ('table', 'view') ORDER BY (name = 'sqlite_sequence'), name");}function
count_tables($i){return
array();}function
table_status($C=""){$J=array();foreach(get_rows("SELECT name AS Name, type AS Engine, 'rowid' AS Oid, '' AS Auto_increment FROM sqlite_master WHERE type IN ('table', 'view') ".($C!=""?"AND name = ".q($C):"ORDER BY name"))as$K){$K["Rows"]=get_val("SELECT COUNT(*) FROM ".idf_escape($K["Name"]));$J[$K["Name"]]=$K;}foreach(get_rows("SELECT * FROM sqlite_sequence".($C!=""?" WHERE name = ".q($C):""),null,"")as$K)$J[$K["name"]]["Auto_increment"]=$K["seq"];return$J;}function
is_view($S){return$S["Engine"]=="view";}function
fk_support($S){return!get_val("SELECT sqlite_compileoption_used('OMIT_FOREIGN_KEY')");}function
fields($R){$J=array();$G="";foreach(get_rows("PRAGMA table_".(min_version(3.31)?"x":"")."info(".table($R).")")as$K){$C=$K["name"];$U=strtolower($K["type"]);$k=$K["dflt_value"];$J[$C]=array("field"=>$C,"type"=>(preg_match('~int~i',$U)?"integer":(preg_match('~char|clob|text~i',$U)?"text":(preg_match('~blob~i',$U)?"blob":(preg_match('~real|floa|doub~i',$U)?"real":"numeric")))),"full_type"=>$U,"default"=>(preg_match("~^'(.*)'$~",$k,$B)?str_replace("''","'",$B[1]):($k=="NULL"?null:$k)),"null"=>!$K["notnull"],"privileges"=>array("select"=>1,"insert"=>1,"update"=>1,"where"=>1,"order"=>1),"primary"=>$K["pk"],);if($K["pk"]){if($G!="")$J[$G]["auto_increment"]=false;elseif(preg_match('~^integer$~i',$U))$J[$C]["auto_increment"]=true;$G=$C;}}$Nh=get_val("SELECT sql FROM sqlite_master WHERE type = 'table' AND name = ".q($R));$u='(("[^"]*+")+|[a-z0-9_]+)';preg_match_all('~'.$u.'\s+text\s+COLLATE\s+(\'[^\']+\'|\S+)~i',$Nh,$Ne,PREG_SET_ORDER);foreach($Ne
as$B){$C=str_replace('""','"',preg_replace('~^"|"$~','',$B[1]));if($J[$C])$J[$C]["collation"]=trim($B[3],"'");}preg_match_all('~'.$u.'\s.*GENERATED ALWAYS AS \((.+)\) (STORED|VIRTUAL)~i',$Nh,$Ne,PREG_SET_ORDER);foreach($Ne
as$B){$C=str_replace('""','"',preg_replace('~^"|"$~','',$B[1]));$J[$C]["default"]=$B[3];$J[$C]["generated"]=strtoupper($B[4]);}return$J;}function
indexes($R,$g=null){$g=connection($g);$J=array();$Nh=get_val("SELECT sql FROM sqlite_master WHERE type = 'table' AND name = ".q($R),0,$g);if(preg_match('~\bPRIMARY\s+KEY\s*\((([^)"]+|"[^"]*"|`[^`]*`)++)~i',$Nh,$B)){$J[""]=array("type"=>"PRIMARY","columns"=>array(),"lengths"=>array(),"descs"=>array());preg_match_all('~((("[^"]*+")+|(?:`[^`]*+`)+)|(\S+))(\s+(ASC|DESC))?(,\s*|$)~i',$B[1],$Ne,PREG_SET_ORDER);foreach($Ne
as$B){$J[""]["columns"][]=idf_unescape($B[2]).$B[4];$J[""]["descs"][]=(preg_match('~DESC~i',$B[5])?'1':null);}}if(!$J){foreach(fields($R)as$C=>$m){if($m["primary"])$J[""]=array("type"=>"PRIMARY","columns"=>array($C),"lengths"=>array(),"descs"=>array(null));}}$Rh=get_key_vals("SELECT name, sql FROM sqlite_master WHERE type = 'index' AND tbl_name = ".q($R),$g);foreach(get_rows("PRAGMA index_list(".table($R).")",$g)as$K){$C=$K["name"];$v=array("type"=>($K["unique"]?"UNIQUE":"INDEX"));$v["lengths"]=array();$v["descs"]=array();foreach(get_rows("PRAGMA index_info(".idf_escape($C).")",$g)as$jh){$v["columns"][]=$jh["name"];$v["descs"][]=null;}if(preg_match('~^CREATE( UNIQUE)? INDEX '.preg_quote(idf_escape($C).' ON '.idf_escape($R),'~').' \((.*)\)$~i',$Rh[$C],$Wg)){preg_match_all('/("[^"]*+")+( DESC)?/',$Wg[2],$Ne);foreach($Ne[2]as$x=>$X){if($X)$v["descs"][$x]='1';}}if(!$J[""]||$v["type"]!="UNIQUE"||$v["columns"]!=$J[""]["columns"]||$v["descs"]!=$J[""]["descs"]||!preg_match("~^sqlite_~",$C))$J[$C]=$v;}return$J;}function
foreign_keys($R){$J=array();foreach(get_rows("PRAGMA foreign_key_list(".table($R).")")as$K){$p=&$J[$K["id"]];if(!$p)$p=$K;$p["source"][]=$K["from"];$p["target"][]=$K["to"];}return$J;}function
view($C){return
array("select"=>preg_replace('~^(?:[^`"[]+|`[^`]*`|"[^"]*")* AS\s+~iU','',get_val("SELECT sql FROM sqlite_master WHERE type = 'view' AND name = ".q($C))));}function
collations(){return(isset($_GET["create"])?get_vals("PRAGMA collation_list",1):array());}function
information_schema($j){return
false;}function
error(){return
h(connection()->error);}function
check_sqlite_name($C){$Nc="db|sdb|sqlite";if(!preg_match("~^[^\\0]*\\.($Nc)\$~",$C)){connection()->error=sprintf('Please use one of the extensions %s.',str_replace("|",", ",$Nc));return
false;}return
true;}function
create_database($j,$c){if(file_exists($j)){connection()->error='File exists.';return
false;}if(!check_sqlite_name($j))return
false;try{$_=new
Db();$_->attach($j,'','');}catch(\Exception$Fc){connection()->error=$Fc->getMessage();return
false;}$_->query('PRAGMA encoding = "UTF-8"');$_->query('CREATE TABLE adminer (i)');$_->query('DROP TABLE adminer');return
true;}function
drop_databases($i){connection()->attach(":memory:",'','');foreach($i
as$j){if(!@unlink($j)){connection()->error='File exists.';return
false;}}return
true;}function
rename_database($C,$c){if(!check_sqlite_name($C))return
false;connection()->attach(":memory:",'','');connection()->error='File exists.';return@rename(DB,$C);}function
auto_increment(){return" PRIMARY KEY AUTOINCREMENT";}function
alter_table($R,$C,$n,$gd,$mb,$vc,$c,$_a,$kg){$fj=($R==""||$gd);foreach($n
as$m){if($m[0]!=""||!$m[1]||$m[2]){$fj=true;break;}}$b=array();$Yf=array();foreach($n
as$m){if($m[1]){$b[]=($fj?$m[1]:"ADD ".implode($m[1]));if($m[0]!="")$Yf[$m[0]]=$m[1][0];}}if(!$fj){foreach($b
as$X){if(!queries("ALTER TABLE ".table($R)." $X"))return
false;}if($R!=$C&&!queries("ALTER TABLE ".table($R)." RENAME TO ".table($C)))return
false;}elseif(!recreate_table($R,$C,$b,$Yf,$gd,$_a))return
false;if($_a){queries("BEGIN");queries("UPDATE sqlite_sequence SET seq = $_a WHERE name = ".q($C));if(!connection()->affected_rows)queries("INSERT INTO sqlite_sequence (name, seq) VALUES (".q($C).", $_a)");queries("COMMIT");}return
true;}function
recreate_table($R,$C,array$n,array$Yf,array$gd,$_a="",$w=array(),$gc="",$ja=""){if($R!=""){if(!$n){foreach(fields($R)as$x=>$m){if($w)$m["auto_increment"]=0;$n[]=process_field($m,$m);$Yf[$x]=idf_escape($x);}}$Bg=false;foreach($n
as$m){if($m[6])$Bg=true;}$ic=array();foreach($w
as$x=>$X){if($X[2]=="DROP"){$ic[$X[1]]=true;unset($w[$x]);}}foreach(indexes($R)as$re=>$v){$e=array();foreach($v["columns"]as$x=>$d){if(!$Yf[$d])continue
2;$e[]=$Yf[$d].($v["descs"][$x]?" DESC":"");}if(!$ic[$re]){if($v["type"]!="PRIMARY"||!$Bg)$w[]=array($v["type"],$re,$e);}}foreach($w
as$x=>$X){if($X[0]=="PRIMARY"){unset($w[$x]);$gd[]="  PRIMARY KEY (".implode(", ",$X[2]).")";}}foreach(foreign_keys($R)as$re=>$p){foreach($p["source"]as$x=>$d){if(!$Yf[$d])continue
2;$p["source"][$x]=idf_unescape($Yf[$d]);}if(!isset($gd[" $re"]))$gd[]=" ".format_foreign_key($p);}queries("BEGIN");}$Ta=array();foreach($n
as$m){if(preg_match('~GENERATED~',$m[3]))unset($Yf[array_search($m[0],$Yf)]);$Ta[]="  ".implode($m);}$Ta=array_merge($Ta,array_filter($gd));foreach(driver()->checkConstraints($R)as$Va){if($Va!=$gc)$Ta[]="  CHECK ($Va)";}if($ja)$Ta[]="  CHECK ($ja)";$qi=($R==$C?"adminer_$C":$C);if(!queries("CREATE TABLE ".table($qi)." (\n".implode(",\n",$Ta)."\n)"))return
false;if($R!=""){if($Yf&&!queries("INSERT INTO ".table($qi)." (".implode(", ",$Yf).") SELECT ".implode(", ",array_map('Adminer\idf_escape',array_keys($Yf)))." FROM ".table($R)))return
false;$Pi=array();foreach(triggers($R)as$Ni=>$xi){$Mi=trigger($Ni,$R);$Pi[]="CREATE TRIGGER ".idf_escape($Ni)." ".implode(" ",$xi)." ON ".table($C)."\n$Mi[Statement]";}$_a=$_a?"":get_val("SELECT seq FROM sqlite_sequence WHERE name = ".q($R));if(!queries("DROP TABLE ".table($R))||($R==$C&&!queries("ALTER TABLE ".table($qi)." RENAME TO ".table($C)))||!alter_indexes($C,$w))return
false;if($_a)queries("UPDATE sqlite_sequence SET seq = $_a WHERE name = ".q($C));foreach($Pi
as$Mi){if(!queries($Mi))return
false;}queries("COMMIT");}return
true;}function
index_sql($R,$U,$C,$e){return"CREATE $U ".($U!="INDEX"?"INDEX ":"").idf_escape($C!=""?$C:uniqid($R."_"))." ON ".table($R)." $e";}function
alter_indexes($R,$b){foreach($b
as$G){if($G[0]=="PRIMARY")return
recreate_table($R,$R,array(),array(),array(),"",$b);}foreach(array_reverse($b)as$X){if(!queries($X[2]=="DROP"?"DROP INDEX ".idf_escape($X[1]):index_sql($R,$X[0],$X[1],"(".implode(", ",$X[2]).")")))return
false;}return
true;}function
truncate_tables($T){return
apply_queries("DELETE FROM",$T);}function
drop_views($rj){return
apply_queries("DROP VIEW",$rj);}function
drop_tables($T){return
apply_queries("DROP TABLE",$T);}function
move_tables($T,$rj,$oi){return
false;}function
trigger($C,$R){if($C=="")return
array("Statement"=>"BEGIN\n\t;\nEND");$u='(?:[^`"\s]+|`[^`]*`|"[^"]*")+';$Oi=trigger_options();preg_match("~^CREATE\\s+TRIGGER\\s*$u\\s*(".implode("|",$Oi["Timing"]).")\\s+([a-z]+)(?:\\s+OF\\s+($u))?\\s+ON\\s*$u\\s*(?:FOR\\s+EACH\\s+ROW\\s)?(.*)~is",get_val("SELECT sql FROM sqlite_master WHERE type = 'trigger' AND name = ".q($C)),$B);$wf=$B[3];return
array("Timing"=>strtoupper($B[1]),"Event"=>strtoupper($B[2]).($wf?" OF":""),"Of"=>idf_unescape($wf),"Trigger"=>$C,"Statement"=>$B[4],);}function
triggers($R){$J=array();$Oi=trigger_options();foreach(get_rows("SELECT * FROM sqlite_master WHERE type = 'trigger' AND tbl_name = ".q($R))as$K){preg_match('~^CREATE\s+TRIGGER\s*(?:[^`"\s]+|`[^`]*`|"[^"]*")+\s*('.implode("|",$Oi["Timing"]).')\s*(.*?)\s+ON\b~i',$K["sql"],$B);$J[$K["name"]]=array($B[1],$B[2]);}return$J;}function
trigger_options(){return
array("Timing"=>array("BEFORE","AFTER","INSTEAD OF"),"Event"=>array("INSERT","UPDATE","UPDATE OF","DELETE"),"Type"=>array("FOR EACH ROW"),);}function
begin(){return
queries("BEGIN");}function
last_id($I){return
get_val("SELECT LAST_INSERT_ROWID()");}function
explain($f,$H){return$f->query("EXPLAIN QUERY PLAN $H");}function
found_rows($S,$Z){}function
types(){return
array();}function
create_sql($R,$_a,$Xh){$J=get_val("SELECT sql FROM sqlite_master WHERE type IN ('table', 'view') AND name = ".q($R));foreach(indexes($R)as$C=>$v){if($C=='')continue;$J
.=";\n\n".index_sql($R,$v['type'],$C,"(".implode(", ",array_map('Adminer\idf_escape',$v['columns'])).")");}return$J;}function
truncate_sql($R){return"DELETE FROM ".table($R);}function
use_sql($Lb){}function
trigger_sql($R){return
implode(get_vals("SELECT sql || ';;\n' FROM sqlite_master WHERE type = 'trigger' AND tbl_name = ".q($R)));}function
show_variables(){$J=array();foreach(get_rows("PRAGMA pragma_list")as$K){$C=$K["name"];if($C!="pragma_list"&&$C!="compile_options"){$J[$C]=array($C,'');foreach(get_rows("PRAGMA $C")as$K)$J[$C][1].=implode(", ",$K)."\n";}}return$J;}function
show_status(){$J=array();foreach(get_vals("PRAGMA compile_options")as$Kf)$J[]=explode("=",$Kf,2)+array('','');return$J;}function
convert_field($m){}function
unconvert_field($m,$J){return$J;}function
support($Sc){return
preg_match('~^(check|columns|database|drop_col|dump|indexes|descidx|move_col|sql|status|table|trigger|variables|view|view_trigger)$~',$Sc);}}add_driver("pgsql","PostgreSQL");if(isset($_GET["pgsql"])){define('Adminer\DRIVER',"pgsql");if(extension_loaded("pgsql")&&$_GET["ext"]!="pdo"){class
PgsqlDb
extends
SqlDb{var$extension="PgSQL";var$timeout=0;private$link,$string,$database=true;function
_error($Ac,$l){if(ini_bool("html_errors"))$l=html_entity_decode(strip_tags($l));$l=preg_replace('~^[^:]*: ~','',$l);$this->error=$l;}function
attach($N,$V,$F){$j=adminer()->database();set_error_handler(array($this,'_error'));$this->string="host='".str_replace(":","' port='",addcslashes($N,"'\\"))."' user='".addcslashes($V,"'\\")."' password='".addcslashes($F,"'\\")."'";$Sh=adminer()->connectSsl();if(isset($Sh["mode"]))$this->string
.=" sslmode='".$Sh["mode"]."'";$this->link=@pg_connect("$this->string dbname='".($j!=""?addcslashes($j,"'\\"):"postgres")."'",PGSQL_CONNECT_FORCE_NEW);if(!$this->link&&$j!=""){$this->database=false;$this->link=@pg_connect("$this->string dbname='postgres'",PGSQL_CONNECT_FORCE_NEW);}restore_error_handler();if($this->link)pg_set_client_encoding($this->link,"UTF8");return($this->link?'':$this->error);}function
quote($Q){return(function_exists('pg_escape_literal')?pg_escape_literal($this->link,$Q):"'".pg_escape_string($this->link,$Q)."'");}function
value($X,array$m){return($m["type"]=="bytea"&&$X!==null?pg_unescape_bytea($X):$X);}function
select_db($Lb){if($Lb==adminer()->database())return$this->database;$J=@pg_connect("$this->string dbname='".addcslashes($Lb,"'\\")."'",PGSQL_CONNECT_FORCE_NEW);if($J)$this->link=$J;return$J;}function
close(){$this->link=@pg_connect("$this->string dbname='postgres'");}function
query($H,$Ti=false){$I=@pg_query($this->link,$H);$this->error="";if(!$I){$this->error=pg_last_error($this->link);$J=false;}elseif(!pg_num_fields($I)){$this->affected_rows=pg_affected_rows($I);$J=true;}else$J=new
Result($I);if($this->timeout){$this->timeout=0;$this->query("RESET statement_timeout");}return$J;}function
warnings(){return
h(pg_last_notice($this->link));}function
copyFrom($R,array$L){$this->error='';set_error_handler(function($Ac,$l){$this->error=(ini_bool('html_errors')?html_entity_decode($l):$l);return
true;});$J=pg_copy_from($this->link,$R,$L);restore_error_handler();return$J;}}class
Result{var$num_rows;private$result,$offset=0;function
__construct($I){$this->result=$I;$this->num_rows=pg_num_rows($I);}function
fetch_assoc(){return
pg_fetch_assoc($this->result);}function
fetch_row(){return
pg_fetch_row($this->result);}function
fetch_field(){$d=$this->offset++;$J=new
\stdClass;$J->orgtable=pg_field_table($this->result,$d);$J->name=pg_field_name($this->result,$d);$J->type=pg_field_type($this->result,$d);$J->charsetnr=($J->type=="bytea"?63:0);return$J;}function
__destruct(){pg_free_result($this->result);}}}elseif(extension_loaded("pdo_pgsql")){class
PgsqlDb
extends
PdoDb{var$extension="PDO_PgSQL";var$timeout=0;function
attach($N,$V,$F){$j=adminer()->database();$kc="pgsql:host='".str_replace(":","' port='",addcslashes($N,"'\\"))."' client_encoding=utf8 dbname='".($j!=""?addcslashes($j,"'\\"):"postgres")."'";$Sh=adminer()->connectSsl();if(isset($Sh["mode"]))$kc
.=" sslmode='".$Sh["mode"]."'";return$this->dsn($kc,$V,$F);}function
select_db($Lb){return(adminer()->database()==$Lb);}function
query($H,$Ti=false){$J=parent::query($H,$Ti);if($this->timeout){$this->timeout=0;parent::query("RESET statement_timeout");}return$J;}function
warnings(){}function
copyFrom($R,array$L){$J=$this->pdo->pgsqlCopyFromArray($R,$L);$this->error=idx($this->pdo->errorInfo(),2)?:'';return$J;}function
close(){}}}if(class_exists('Adminer\PgsqlDb')){class
Db
extends
PgsqlDb{function
multi_query($H){if(preg_match('~\bCOPY\s+(.+?)\s+FROM\s+stdin;\n?(.*)\n\\\\\.$~is',str_replace("\r\n","\n",$H),$B)){$L=explode("\n",$B[2]);$this->affected_rows=count($L);return$this->copyFrom($B[1],$L);}return
parent::multi_query($H);}}}class
Driver
extends
SqlDriver{static$Nc=array("PgSQL","PDO_PgSQL");static$pe="pgsql";var$operators=array("=","<",">","<=",">=","!=","~","!~","LIKE","LIKE %%","ILIKE","ILIKE %%","IN","IS NULL","NOT LIKE","NOT IN","IS NOT NULL");var$functions=array("char_length","lower","round","to_hex","to_timestamp","upper");var$grouping=array("avg","count","count distinct","max","min","sum");static
function
connect($N,$V,$F){$f=parent::connect($N,$V,$F);if(is_string($f))return$f;$pj=get_val("SELECT version()",0,$f);$f->flavor=(preg_match('~CockroachDB~',$pj)?'cockroach':'');$f->server_info=preg_replace('~^\D*([\d.]+[-\w]*).*~','\1',$pj);if(min_version(9,0,$f))$f->query("SET application_name = 'Adminer'");if($f->flavor=='cockroach')add_driver(DRIVER,"CockroachDB");return$f;}function
__construct(Db$f){parent::__construct($f);$this->types=array('Numbers'=>array("smallint"=>5,"integer"=>10,"bigint"=>19,"boolean"=>1,"numeric"=>0,"real"=>7,"double precision"=>16,"money"=>20),'Date and time'=>array("date"=>13,"time"=>17,"timestamp"=>20,"timestamptz"=>21,"interval"=>0),'Strings'=>array("character"=>0,"character varying"=>0,"text"=>0,"tsquery"=>0,"tsvector"=>0,"uuid"=>0,"xml"=>0),'Binary'=>array("bit"=>0,"bit varying"=>0,"bytea"=>0),'Network'=>array("cidr"=>43,"inet"=>43,"macaddr"=>17,"macaddr8"=>23,"txid_snapshot"=>0),'Geometry'=>array("box"=>0,"circle"=>0,"line"=>0,"lseg"=>0,"path"=>0,"point"=>0,"polygon"=>0),);if(min_version(9.2,0,$f)){$this->types['Strings']["json"]=4294967295;if(min_version(9.4,0,$f))$this->types['Strings']["jsonb"]=4294967295;}$this->insertFunctions=array("char"=>"md5","date|time"=>"now",);$this->editFunctions=array(number_type()=>"+/-","date|time"=>"+ interval/- interval","char|text"=>"||",);if(min_version(12,0,$f))$this->generated=array("STORED");}function
enumLength(array$m){$xc=$this->types['User types'][$m["type"]];return($xc?type_values($xc):"");}function
setUserTypes($Si){$this->types['User types']=array_flip($Si);}function
insertReturning($R){$_a=array_filter(fields($R),function($m){return$m['auto_increment'];});return(count($_a)==1?" RETURNING ".idf_escape(key($_a)):"");}function
insertUpdate($R,array$L,array$G){foreach($L
as$O){$bj=array();$Z=array();foreach($O
as$x=>$X){$bj[]="$x = $X";if(isset($G[idf_unescape($x)]))$Z[]="$x = $X";}if(!(($Z&&queries("UPDATE ".table($R)." SET ".implode(", ",$bj)." WHERE ".implode(" AND ",$Z))&&connection()->affected_rows)||queries("INSERT INTO ".table($R)." (".implode(", ",array_keys($O)).") VALUES (".implode(", ",$O).")")))return
false;}return
true;}function
slowQuery($H,$wi){$this->conn->query("SET statement_timeout = ".(1000*$wi));$this->conn->timeout=1000*$wi;return$H;}function
convertSearch($u,array$X,array$m){$ti="char|text";if(strpos($X["op"],"LIKE")===false)$ti
.="|date|time(stamp)?|boolean|uuid|inet|cidr|macaddr|".number_type();return(preg_match("~$ti~",$m["type"])?$u:"CAST($u AS text)");}function
quoteBinary($kh){return"'\\x".bin2hex($kh)."'";}function
warnings(){return$this->conn->warnings();}function
tableHelp($C,$ne=false){$Ge=array("information_schema"=>"infoschema","pg_catalog"=>($ne?"view":"catalog"),);$_=$Ge[$_GET["ns"]];if($_)return"$_-".str_replace("_","-",$C).".html";}function
supportsIndex(array$S){return$S["Engine"]!="view";}function
hasCStyleEscapes(){static$Pa;if($Pa===null)$Pa=(get_val("SHOW standard_conforming_strings",0,$this->conn)=="off");return$Pa;}}function
idf_escape($u){return'"'.str_replace('"','""',$u).'"';}function
table($u){return
idf_escape($u);}function
get_databases($ed){return
get_vals("SELECT datname FROM pg_database
WHERE datallowconn = TRUE AND has_database_privilege(datname, 'CONNECT')
ORDER BY datname");}function
limit($H,$Z,$z,$D=0,$xh=" "){return" $H$Z".($z?$xh."LIMIT $z".($D?" OFFSET $D":""):"");}function
limit1($R,$H,$Z,$xh="\n"){return(preg_match('~^INTO~',$H)?limit($H,$Z,1,0,$xh):" $H".(is_view(table_status1($R))?$Z:$xh."WHERE ctid = (SELECT ctid FROM ".table($R).$Z.$xh."LIMIT 1)"));}function
db_collation($j,$hb){return
get_val("SELECT datcollate FROM pg_database WHERE datname = ".q($j));}function
logged_user(){return
get_val("SELECT user");}function
tables_list(){$H="SELECT table_name, table_type FROM information_schema.tables WHERE table_schema = current_schema()";if(support("materializedview"))$H
.="
UNION ALL
SELECT matviewname, 'MATERIALIZED VIEW'
FROM pg_matviews
WHERE schemaname = current_schema()";$H
.="
ORDER BY 1";return
get_key_vals($H);}function
count_tables($i){$J=array();foreach($i
as$j){if(connection()->select_db($j))$J[$j]=count(tables_list());}return$J;}function
table_status($C=""){static$Cd;if($Cd===null)$Cd=get_val("SELECT 'pg_table_size'::regproc");$J=array();foreach(get_rows("SELECT
	c.relname AS \"Name\",
	CASE c.relkind WHEN 'r' THEN 'table' WHEN 'm' THEN 'materialized view' ELSE 'view' END AS \"Engine\"".($Cd?",
	pg_table_size(c.oid) AS \"Data_length\",
	pg_indexes_size(c.oid) AS \"Index_length\"":"").",
	obj_description(c.oid, 'pg_class') AS \"Comment\",
	".(min_version(12)?"''":"CASE WHEN c.relhasoids THEN 'oid' ELSE '' END")." AS \"Oid\",
	c.reltuples as \"Rows\",
	n.nspname
FROM pg_class c
JOIN pg_namespace n ON(n.nspname = current_schema() AND n.oid = c.relnamespace)
WHERE relkind IN ('r', 'm', 'v', 'f', 'p')
".($C!=""?"AND relname = ".q($C):"ORDER BY relname"))as$K)$J[$K["Name"]]=$K;return$J;}function
is_view($S){return
in_array($S["Engine"],array("view","materialized view"));}function
fk_support($S){return
true;}function
fields($R){$J=array();$ra=array('timestamp without time zone'=>'timestamp','timestamp with time zone'=>'timestamptz',);foreach(get_rows("SELECT
	a.attname AS field,
	format_type(a.atttypid, a.atttypmod) AS full_type,
	pg_get_expr(d.adbin, d.adrelid) AS default,
	a.attnotnull::int,
	col_description(c.oid, a.attnum) AS comment".(min_version(10)?",
	a.attidentity".(min_version(12)?",
	a.attgenerated":""):"")."
FROM pg_class c
JOIN pg_namespace n ON c.relnamespace = n.oid
JOIN pg_attribute a ON c.oid = a.attrelid
LEFT JOIN pg_attrdef d ON c.oid = d.adrelid AND a.attnum = d.adnum
WHERE c.relname = ".q($R)."
AND n.nspname = current_schema()
AND NOT a.attisdropped
AND a.attnum > 0
ORDER BY a.attnum")as$K){preg_match('~([^([]+)(\((.*)\))?([a-z ]+)?((\[[0-9]*])*)$~',$K["full_type"],$B);list(,$U,$y,$K["length"],$ka,$va)=$B;$K["length"].=$va;$Xa=$U.$ka;if(isset($ra[$Xa])){$K["type"]=$ra[$Xa];$K["full_type"]=$K["type"].$y.$va;}else{$K["type"]=$U;$K["full_type"]=$K["type"].$y.$ka.$va;}if(in_array($K['attidentity'],array('a','d')))$K['default']='GENERATED '.($K['attidentity']=='d'?'BY DEFAULT':'ALWAYS').' AS IDENTITY';$K["generated"]=($K["attgenerated"]=="s"?"STORED":"");$K["null"]=!$K["attnotnull"];$K["auto_increment"]=$K['attidentity']||preg_match('~^nextval\(~i',$K["default"])||preg_match('~^unique_rowid\(~',$K["default"]);$K["privileges"]=array("insert"=>1,"select"=>1,"update"=>1,"where"=>1,"order"=>1);if(preg_match('~(.+)::[^,)]+(.*)~',$K["default"],$B))$K["default"]=($B[1]=="NULL"?null:idf_unescape($B[1]).$B[2]);$J[$K["field"]]=$K;}return$J;}function
indexes($R,$g=null){$g=connection($g);$J=array();$gi=get_val("SELECT oid FROM pg_class WHERE relnamespace = (SELECT oid FROM pg_namespace WHERE nspname = current_schema()) AND relname = ".q($R),0,$g);$e=get_key_vals("SELECT attnum, attname FROM pg_attribute WHERE attrelid = $gi AND attnum > 0",$g);foreach(get_rows("SELECT relname, indisunique::int, indisprimary::int, indkey, indoption, (indpred IS NOT NULL)::int as indispartial
FROM pg_index i, pg_class ci
WHERE i.indrelid = $gi AND ci.oid = i.indexrelid
ORDER BY indisprimary DESC, indisunique DESC",$g)as$K){$Xg=$K["relname"];$J[$Xg]["type"]=($K["indispartial"]?"INDEX":($K["indisprimary"]?"PRIMARY":($K["indisunique"]?"UNIQUE":"INDEX")));$J[$Xg]["columns"]=array();$J[$Xg]["descs"]=array();if($K["indkey"]){foreach(explode(" ",$K["indkey"])as$Xd)$J[$Xg]["columns"][]=$e[$Xd];foreach(explode(" ",$K["indoption"])as$Yd)$J[$Xg]["descs"][]=(intval($Yd)&1?'1':null);}$J[$Xg]["lengths"]=array();}return$J;}function
foreign_keys($R){$J=array();foreach(get_rows("SELECT conname, condeferrable::int AS deferrable, pg_get_constraintdef(oid) AS definition
FROM pg_constraint
WHERE conrelid = (SELECT pc.oid FROM pg_class AS pc INNER JOIN pg_namespace AS pn ON (pn.oid = pc.relnamespace) WHERE pc.relname = ".q($R)." AND pn.nspname = current_schema())
AND contype = 'f'::char
ORDER BY conkey, conname")as$K){if(preg_match('~FOREIGN KEY\s*\((.+)\)\s*REFERENCES (.+)\((.+)\)(.*)$~iA',$K['definition'],$B)){$K['source']=array_map('Adminer\idf_unescape',array_map('trim',explode(',',$B[1])));if(preg_match('~^(("([^"]|"")+"|[^"]+)\.)?"?("([^"]|"")+"|[^"]+)$~',$B[2],$Le)){$K['ns']=idf_unescape($Le[2]);$K['table']=idf_unescape($Le[4]);}$K['target']=array_map('Adminer\idf_unescape',array_map('trim',explode(',',$B[3])));$K['on_delete']=(preg_match("~ON DELETE (".driver()->onActions.")~",$B[4],$Le)?$Le[1]:'NO ACTION');$K['on_update']=(preg_match("~ON UPDATE (".driver()->onActions.")~",$B[4],$Le)?$Le[1]:'NO ACTION');$J[$K['conname']]=$K;}}return$J;}function
view($C){return
array("select"=>trim(get_val("SELECT pg_get_viewdef(".get_val("SELECT oid FROM pg_class WHERE relnamespace = (SELECT oid FROM pg_namespace WHERE nspname = current_schema()) AND relname = ".q($C)).")")));}function
collations(){return
array();}function
information_schema($j){return
get_schema()=="information_schema";}function
error(){$J=h(connection()->error);if(preg_match('~^(.*\n)?([^\n]*)\n( *)\^(\n.*)?$~s',$J,$B))$J=$B[1].preg_replace('~((?:[^&]|&[^;]*;){'.strlen($B[3]).'})(.*)~','\1<b>\2</b>',$B[2]).$B[4];return
nl_br($J);}function
create_database($j,$c){return
queries("CREATE DATABASE ".idf_escape($j).($c?" ENCODING ".idf_escape($c):""));}function
drop_databases($i){connection()->close();return
apply_queries("DROP DATABASE",$i,'Adminer\idf_escape');}function
rename_database($C,$c){connection()->close();return
queries("ALTER DATABASE ".idf_escape(DB)." RENAME TO ".idf_escape($C));}function
auto_increment(){return"";}function
alter_table($R,$C,$n,$gd,$mb,$vc,$c,$_a,$kg){$b=array();$Kg=array();if($R!=""&&$R!=$C)$Kg[]="ALTER TABLE ".table($R)." RENAME TO ".table($C);$yh="";foreach($n
as$m){$d=idf_escape($m[0]);$X=$m[1];if(!$X)$b[]="DROP $d";else{$lj=$X[5];unset($X[5]);if($m[0]==""){if(isset($X[6]))$X[1]=($X[1]==" bigint"?" big":($X[1]==" smallint"?" small":" "))."serial";$b[]=($R!=""?"ADD ":"  ").implode($X);if(isset($X[6]))$b[]=($R!=""?"ADD":" ")." PRIMARY KEY ($X[0])";}else{if($d!=$X[0])$Kg[]="ALTER TABLE ".table($C)." RENAME $d TO $X[0]";$b[]="ALTER $d TYPE$X[1]";$zh=$R."_".idf_unescape($X[0])."_seq";$b[]="ALTER $d ".($X[3]?"SET".preg_replace('~GENERATED ALWAYS(.*) STORED~','EXPRESSION\1',$X[3]):(isset($X[6])?"SET DEFAULT nextval(".q($zh).")":"DROP DEFAULT"));if(isset($X[6]))$yh="CREATE SEQUENCE IF NOT EXISTS ".idf_escape($zh)." OWNED BY ".idf_escape($R).".$X[0]";$b[]="ALTER $d ".($X[2]==" NULL"?"DROP NOT":"SET").$X[2];}if($m[0]!=""||$lj!="")$Kg[]="COMMENT ON COLUMN ".table($C).".$X[0] IS ".($lj!=""?substr($lj,9):"''");}}$b=array_merge($b,$gd);if($R=="")array_unshift($Kg,"CREATE TABLE ".table($C)." (\n".implode(",\n",$b)."\n)");elseif($b)array_unshift($Kg,"ALTER TABLE ".table($R)."\n".implode(",\n",$b));if($yh)array_unshift($Kg,$yh);if($mb!==null)$Kg[]="COMMENT ON TABLE ".table($C)." IS ".q($mb);foreach($Kg
as$H){if(!queries($H))return
false;}return
true;}function
alter_indexes($R,$b){$h=array();$fc=array();$Kg=array();foreach($b
as$X){if($X[0]!="INDEX")$h[]=($X[2]=="DROP"?"\nDROP CONSTRAINT ".idf_escape($X[1]):"\nADD".($X[1]!=""?" CONSTRAINT ".idf_escape($X[1]):"")." $X[0] ".($X[0]=="PRIMARY"?"KEY ":"")."(".implode(", ",$X[2]).")");elseif($X[2]=="DROP")$fc[]=idf_escape($X[1]);else$Kg[]="CREATE INDEX ".idf_escape($X[1]!=""?$X[1]:uniqid($R."_"))." ON ".table($R)." (".implode(", ",$X[2]).")";}if($h)array_unshift($Kg,"ALTER TABLE ".table($R).implode(",",$h));if($fc)array_unshift($Kg,"DROP INDEX ".implode(", ",$fc));foreach($Kg
as$H){if(!queries($H))return
false;}return
true;}function
truncate_tables($T){return
queries("TRUNCATE ".implode(", ",array_map('Adminer\table',$T)));}function
drop_views($rj){return
drop_tables($rj);}function
drop_tables($T){foreach($T
as$R){$P=table_status1($R);if(!queries("DROP ".strtoupper($P["Engine"])." ".table($R)))return
false;}return
true;}function
move_tables($T,$rj,$oi){foreach(array_merge($T,$rj)as$R){$P=table_status1($R);if(!queries("ALTER ".strtoupper($P["Engine"])." ".table($R)." SET SCHEMA ".idf_escape($oi)))return
false;}return
true;}function
trigger($C,$R){if($C=="")return
array("Statement"=>"EXECUTE PROCEDURE ()");$e=array();$Z="WHERE trigger_schema = current_schema() AND event_object_table = ".q($R)." AND trigger_name = ".q($C);foreach(get_rows("SELECT * FROM information_schema.triggered_update_columns $Z")as$K)$e[]=$K["event_object_column"];$J=array();foreach(get_rows('SELECT trigger_name AS "Trigger", action_timing AS "Timing", event_manipulation AS "Event", \'FOR EACH \' || action_orientation AS "Type", action_statement AS "Statement"
FROM information_schema.triggers'."
$Z
ORDER BY event_manipulation DESC")as$K){if($e&&$K["Event"]=="UPDATE")$K["Event"].=" OF";$K["Of"]=implode(", ",$e);if($J)$K["Event"].=" OR $J[Event]";$J=$K;}return$J;}function
triggers($R){$J=array();foreach(get_rows("SELECT * FROM information_schema.triggers WHERE trigger_schema = current_schema() AND event_object_table = ".q($R))as$K){$Mi=trigger($K["trigger_name"],$R);$J[$Mi["Trigger"]]=array($Mi["Timing"],$Mi["Event"]);}return$J;}function
trigger_options(){return
array("Timing"=>array("BEFORE","AFTER"),"Event"=>array("INSERT","UPDATE","UPDATE OF","DELETE","INSERT OR UPDATE","INSERT OR UPDATE OF","DELETE OR INSERT","DELETE OR UPDATE","DELETE OR UPDATE OF","DELETE OR INSERT OR UPDATE","DELETE OR INSERT OR UPDATE OF"),"Type"=>array("FOR EACH ROW","FOR EACH STATEMENT"),);}function
routine($C,$U){$L=get_rows('SELECT routine_definition AS definition, LOWER(external_language) AS language, *
FROM information_schema.routines
WHERE routine_schema = current_schema() AND specific_name = '.q($C));$J=idx($L,0,array());$J["returns"]=array("type"=>$J["type_udt_name"]);$J["fields"]=get_rows('SELECT parameter_name AS field, data_type AS type, character_maximum_length AS length, parameter_mode AS inout
FROM information_schema.parameters
WHERE specific_schema = current_schema() AND specific_name = '.q($C).'
ORDER BY ordinal_position');return$J;}function
routines(){return
get_rows('SELECT specific_name AS "SPECIFIC_NAME", routine_type AS "ROUTINE_TYPE", routine_name AS "ROUTINE_NAME", type_udt_name AS "DTD_IDENTIFIER"
FROM information_schema.routines
WHERE routine_schema = current_schema()
ORDER BY SPECIFIC_NAME');}function
routine_languages(){return
get_vals("SELECT LOWER(lanname) FROM pg_catalog.pg_language");}function
routine_id($C,$K){$J=array();foreach($K["fields"]as$m){$y=$m["length"];$J[]=$m["type"].($y?"($y)":"");}return
idf_escape($C)."(".implode(", ",$J).")";}function
last_id($I){$K=(is_object($I)?$I->fetch_row():array());return($K?$K[0]:0);}function
explain($f,$H){return$f->query("EXPLAIN $H");}function
found_rows($S,$Z){if(preg_match("~ rows=([0-9]+)~",get_val("EXPLAIN SELECT * FROM ".idf_escape($S["Name"]).($Z?" WHERE ".implode(" AND ",$Z):"")),$Wg))return$Wg[1];}function
types(){return
get_key_vals("SELECT oid, typname
FROM pg_type
WHERE typnamespace = (SELECT oid FROM pg_namespace WHERE nspname = current_schema())
AND typtype IN ('b','d','e')
AND typelem = 0");}function
type_values($t){$_c=get_vals("SELECT enumlabel FROM pg_enum WHERE enumtypid = $t ORDER BY enumsortorder");return($_c?"'".implode("', '",array_map('addslashes',$_c))."'":"");}function
schemas(){return
get_vals("SELECT nspname FROM pg_namespace ORDER BY nspname");}function
get_schema(){return
get_val("SELECT current_schema()");}function
set_schema($mh,$g=null){if(!$g)$g=connection();$J=$g->query("SET search_path TO ".idf_escape($mh));driver()->setUserTypes(types());return$J;}function
foreign_keys_sql($R){$J="";$P=table_status1($R);$cd=foreign_keys($R);ksort($cd);foreach($cd
as$bd=>$ad)$J
.="ALTER TABLE ONLY ".idf_escape($P['nspname']).".".idf_escape($P['Name'])." ADD CONSTRAINT ".idf_escape($bd)." $ad[definition] ".($ad['deferrable']?'DEFERRABLE':'NOT DEFERRABLE').";\n";return($J?"$J\n":$J);}function
create_sql($R,$_a,$Xh){$ch=array();$_h=array();$P=table_status1($R);if(is_view($P)){$qj=view($R);return
rtrim("CREATE VIEW ".idf_escape($R)." AS $qj[select]",";");}$n=fields($R);if(count($P)<2||empty($n))return
false;$J="CREATE TABLE ".idf_escape($P['nspname']).".".idf_escape($P['Name'])." (\n    ";foreach($n
as$m){$hg=idf_escape($m['field']).' '.$m['full_type'].default_value($m).($m['null']?"":" NOT NULL");$ch[]=$hg;if(preg_match('~nextval\(\'([^\']+)\'\)~',$m['default'],$Ne)){$zh=$Ne[1];$Mh=first(get_rows((min_version(10)?"SELECT *, cache_size AS cache_value FROM pg_sequences WHERE schemaname = current_schema() AND sequencename = ".q(idf_unescape($zh)):"SELECT * FROM $zh"),null,"-- "));$_h[]=($Xh=="DROP+CREATE"?"DROP SEQUENCE IF EXISTS $zh;\n":"")."CREATE SEQUENCE $zh INCREMENT $Mh[increment_by] MINVALUE $Mh[min_value] MAXVALUE $Mh[max_value]".($_a&&$Mh['last_value']?" START ".($Mh["last_value"]+1):"")." CACHE $Mh[cache_value];";}}if(!empty($_h))$J=implode("\n\n",$_h)."\n\n$J";$G="";foreach(indexes($R)as$Vd=>$v){if($v['type']=='PRIMARY'){$G=$Vd;$ch[]="CONSTRAINT ".idf_escape($Vd)." PRIMARY KEY (".implode(', ',array_map('Adminer\idf_escape',$v['columns'])).")";}}foreach(driver()->checkConstraints($R)as$sb=>$ub)$ch[]="CONSTRAINT ".idf_escape($sb)." CHECK $ub";$J
.=implode(",\n    ",$ch)."\n) WITH (oids = ".($P['Oid']?'true':'false').");";if($P['Comment'])$J
.="\n\nCOMMENT ON TABLE ".idf_escape($P['nspname']).".".idf_escape($P['Name'])." IS ".q($P['Comment']).";";foreach($n
as$Uc=>$m){if($m['comment'])$J
.="\n\nCOMMENT ON COLUMN ".idf_escape($P['nspname']).".".idf_escape($P['Name']).".".idf_escape($Uc)." IS ".q($m['comment']).";";}foreach(get_rows("SELECT indexdef FROM pg_catalog.pg_indexes WHERE schemaname = current_schema() AND tablename = ".q($R).($G?" AND indexname != ".q($G):""),null,"-- ")as$K)$J
.="\n\n$K[indexdef];";return
rtrim($J,';');}function
truncate_sql($R){return"TRUNCATE ".table($R);}function
trigger_sql($R){$P=table_status1($R);$J="";foreach(triggers($R)as$Li=>$Ki){$Mi=trigger($Li,$P['Name']);$J
.="\nCREATE TRIGGER ".idf_escape($Mi['Trigger'])." $Mi[Timing] $Mi[Event] ON ".idf_escape($P["nspname"]).".".idf_escape($P['Name'])." $Mi[Type] $Mi[Statement];;\n";}return$J;}function
use_sql($Lb){return"\connect ".idf_escape($Lb);}function
show_variables(){return
get_rows("SHOW ALL");}function
process_list(){return
get_rows("SELECT * FROM pg_stat_activity ORDER BY ".(min_version(9.2)?"pid":"procpid"));}function
convert_field($m){}function
unconvert_field($m,$J){return$J;}function
support($Sc){return
preg_match('~^(check|database|table|columns|sql|indexes|descidx|comment|view|'.(min_version(9.3)?'materializedview|':'').'scheme|'.(min_version(11)?'procedure|':'').'routine|sequence|trigger|type|variables|drop_col'.(connection()->flavor=='cockroach'?'':'|processlist').'|kill|dump)$~',$Sc);}function
kill_process($X){return
queries("SELECT pg_terminate_backend(".number($X).")");}function
connection_id(){return"SELECT pg_backend_pid()";}function
max_connections(){return
get_val("SHOW max_connections");}}add_driver("oracle","Oracle (beta)");if(isset($_GET["oracle"])){define('Adminer\DRIVER',"oracle");if(extension_loaded("oci8")&&$_GET["ext"]!="pdo"){class
Db
extends
SqlDb{var$extension="oci8";var$_current_db;private$link;function
_error($Ac,$l){if(ini_bool("html_errors"))$l=html_entity_decode(strip_tags($l));$l=preg_replace('~^[^:]*: ~','',$l);$this->error=$l;}function
attach($N,$V,$F){$this->link=@oci_new_connect($V,$F,$N,"AL32UTF8");if($this->link){$this->server_info=oci_server_version($this->link);return'';}$l=oci_error();return$l["message"];}function
quote($Q){return"'".str_replace("'","''",$Q)."'";}function
select_db($Lb){$this->_current_db=$Lb;return
true;}function
query($H,$Ti=false){$I=oci_parse($this->link,$H);$this->error="";if(!$I){$l=oci_error($this->link);$this->errno=$l["code"];$this->error=$l["message"];return
false;}set_error_handler(array($this,'_error'));$J=@oci_execute($I);restore_error_handler();if($J){if(oci_num_fields($I))return
new
Result($I);$this->affected_rows=oci_num_rows($I);oci_free_statement($I);}return$J;}}class
Result{var$num_rows;private$result,$offset=1;function
__construct($I){$this->result=$I;}private
function
convert($K){foreach((array)$K
as$x=>$X){if(is_a($X,'OCILob')||is_a($X,'OCI-Lob'))$K[$x]=$X->load();}return$K;}function
fetch_assoc(){return$this->convert(oci_fetch_assoc($this->result));}function
fetch_row(){return$this->convert(oci_fetch_row($this->result));}function
fetch_field(){$d=$this->offset++;$J=new
\stdClass;$J->name=oci_field_name($this->result,$d);$J->type=oci_field_type($this->result,$d);$J->charsetnr=(preg_match("~raw|blob|bfile~",$J->type)?63:0);return$J;}function
__destruct(){oci_free_statement($this->result);}}}elseif(extension_loaded("pdo_oci")){class
Db
extends
PdoDb{var$extension="PDO_OCI";var$_current_db;function
attach($N,$V,$F){return$this->dsn("oci:dbname=//$N;charset=AL32UTF8",$V,$F);}function
select_db($Lb){$this->_current_db=$Lb;return
true;}}}class
Driver
extends
SqlDriver{static$Nc=array("OCI8","PDO_OCI");static$pe="oracle";var$insertFunctions=array("date"=>"current_date","timestamp"=>"current_timestamp",);var$editFunctions=array("number|float|double"=>"+/-","date|timestamp"=>"+ interval/- interval","char|clob"=>"||",);var$operators=array("=","<",">","<=",">=","!=","LIKE","LIKE %%","IN","IS NULL","NOT LIKE","NOT IN","IS NOT NULL","SQL");var$functions=array("length","lower","round","upper");var$grouping=array("avg","count","count distinct","max","min","sum");function
__construct(Db$f){parent::__construct($f);$this->types=array('Numbers'=>array("number"=>38,"binary_float"=>12,"binary_double"=>21),'Date and time'=>array("date"=>10,"timestamp"=>29,"interval year"=>12,"interval day"=>28),'Strings'=>array("char"=>2000,"varchar2"=>4000,"nchar"=>2000,"nvarchar2"=>4000,"clob"=>4294967295,"nclob"=>4294967295),'Binary'=>array("raw"=>2000,"long raw"=>2147483648,"blob"=>4294967295,"bfile"=>4294967296),);}function
begin(){return
true;}function
insertUpdate($R,array$L,array$G){foreach($L
as$O){$bj=array();$Z=array();foreach($O
as$x=>$X){$bj[]="$x = $X";if(isset($G[idf_unescape($x)]))$Z[]="$x = $X";}if(!(($Z&&queries("UPDATE ".table($R)." SET ".implode(", ",$bj)." WHERE ".implode(" AND ",$Z))&&connection()->affected_rows)||queries("INSERT INTO ".table($R)." (".implode(", ",array_keys($O)).") VALUES (".implode(", ",$O).")")))return
false;}return
true;}function
hasCStyleEscapes(){return
true;}}function
idf_escape($u){return'"'.str_replace('"','""',$u).'"';}function
table($u){return
idf_escape($u);}function
get_databases($ed){return
get_vals("SELECT DISTINCT tablespace_name FROM (
SELECT tablespace_name FROM user_tablespaces
UNION SELECT tablespace_name FROM all_tables WHERE tablespace_name IS NOT NULL
)
ORDER BY 1");}function
limit($H,$Z,$z,$D=0,$xh=" "){return($D?" * FROM (SELECT t.*, rownum AS rnum FROM (SELECT $H$Z) t WHERE rownum <= ".($z+$D).") WHERE rnum > $D":($z?" * FROM (SELECT $H$Z) WHERE rownum <= ".($z+$D):" $H$Z"));}function
limit1($R,$H,$Z,$xh="\n"){return" $H$Z";}function
db_collation($j,$hb){return
get_val("SELECT value FROM nls_database_parameters WHERE parameter = 'NLS_CHARACTERSET'");}function
logged_user(){return
get_val("SELECT USER FROM DUAL");}function
get_current_db(){$j=connection()->_current_db?:DB;unset(connection()->_current_db);return$j;}function
where_owner($_g,$bg="owner"){if(!$_GET["ns"])return'';return"$_g$bg = sys_context('USERENV', 'CURRENT_SCHEMA')";}function
views_table($e){$bg=where_owner('');return"(SELECT $e FROM all_views WHERE ".($bg?:"rownum < 0").")";}function
tables_list(){$qj=views_table("view_name");$bg=where_owner(" AND ");return
get_key_vals("SELECT table_name, 'table' FROM all_tables WHERE tablespace_name = ".q(DB)."$bg
UNION SELECT view_name, 'view' FROM $qj
ORDER BY 1");}function
count_tables($i){$J=array();foreach($i
as$j)$J[$j]=get_val("SELECT COUNT(*) FROM all_tables WHERE tablespace_name = ".q($j));return$J;}function
table_status($C=""){$J=array();$qh=q($C);$j=get_current_db();$qj=views_table("view_name");$bg=where_owner(" AND ");foreach(get_rows('SELECT table_name "Name", \'table\' "Engine", avg_row_len * num_rows "Data_length", num_rows "Rows" FROM all_tables WHERE tablespace_name = '.q($j).$bg.($C!=""?" AND table_name = $qh":"")."
UNION SELECT view_name, 'view', 0, 0 FROM $qj".($C!=""?" WHERE view_name = $qh":"")."
ORDER BY 1")as$K)$J[$K["Name"]]=$K;return$J;}function
is_view($S){return$S["Engine"]=="view";}function
fk_support($S){return
true;}function
fields($R){$J=array();$bg=where_owner(" AND ");foreach(get_rows("SELECT * FROM all_tab_columns WHERE table_name = ".q($R)."$bg ORDER BY column_id")as$K){$U=$K["DATA_TYPE"];$y="$K[DATA_PRECISION],$K[DATA_SCALE]";if($y==",")$y=$K["CHAR_COL_DECL_LENGTH"];$J[$K["COLUMN_NAME"]]=array("field"=>$K["COLUMN_NAME"],"full_type"=>$U.($y?"($y)":""),"type"=>strtolower($U),"length"=>$y,"default"=>$K["DATA_DEFAULT"],"null"=>($K["NULLABLE"]=="Y"),"privileges"=>array("insert"=>1,"select"=>1,"update"=>1,"where"=>1,"order"=>1),);}return$J;}function
indexes($R,$g=null){$J=array();$bg=where_owner(" AND ","aic.table_owner");foreach(get_rows("SELECT aic.*, ac.constraint_type, atc.data_default
FROM all_ind_columns aic
LEFT JOIN all_constraints ac ON aic.index_name = ac.constraint_name AND aic.table_name = ac.table_name AND aic.index_owner = ac.owner
LEFT JOIN all_tab_cols atc ON aic.column_name = atc.column_name AND aic.table_name = atc.table_name AND aic.index_owner = atc.owner
WHERE aic.table_name = ".q($R)."$bg
ORDER BY ac.constraint_type, aic.column_position",$g)as$K){$Vd=$K["INDEX_NAME"];$jb=$K["DATA_DEFAULT"];$jb=($jb?trim($jb,'"'):$K["COLUMN_NAME"]);$J[$Vd]["type"]=($K["CONSTRAINT_TYPE"]=="P"?"PRIMARY":($K["CONSTRAINT_TYPE"]=="U"?"UNIQUE":"INDEX"));$J[$Vd]["columns"][]=$jb;$J[$Vd]["lengths"][]=($K["CHAR_LENGTH"]&&$K["CHAR_LENGTH"]!=$K["COLUMN_LENGTH"]?$K["CHAR_LENGTH"]:null);$J[$Vd]["descs"][]=($K["DESCEND"]&&$K["DESCEND"]=="DESC"?'1':null);}return$J;}function
view($C){$qj=views_table("view_name, text");$L=get_rows('SELECT text "select" FROM '.$qj.' WHERE view_name = '.q($C));return
reset($L);}function
collations(){return
array();}function
information_schema($j){return
get_schema()=="INFORMATION_SCHEMA";}function
error(){return
h(connection()->error);}function
explain($f,$H){$f->query("EXPLAIN PLAN FOR $H");return$f->query("SELECT * FROM plan_table");}function
found_rows($S,$Z){}function
auto_increment(){return"";}function
alter_table($R,$C,$n,$gd,$mb,$vc,$c,$_a,$kg){$b=$fc=array();$Uf=($R?fields($R):array());foreach($n
as$m){$X=$m[1];if($X&&$m[0]!=""&&idf_escape($m[0])!=$X[0])queries("ALTER TABLE ".table($R)." RENAME COLUMN ".idf_escape($m[0])." TO $X[0]");$Tf=$Uf[$m[0]];if($X&&$Tf){$yf=process_field($Tf,$Tf);if($X[2]==$yf[2])$X[2]="";}if($X)$b[]=($R!=""?($m[0]!=""?"MODIFY (":"ADD ("):"  ").implode($X).($R!=""?")":"");else$fc[]=idf_escape($m[0]);}if($R=="")return
queries("CREATE TABLE ".table($C)." (\n".implode(",\n",$b)."\n)");return(!$b||queries("ALTER TABLE ".table($R)."\n".implode("\n",$b)))&&(!$fc||queries("ALTER TABLE ".table($R)." DROP (".implode(", ",$fc).")"))&&($R==$C||queries("ALTER TABLE ".table($R)." RENAME TO ".table($C)));}function
alter_indexes($R,$b){$fc=array();$Kg=array();foreach($b
as$X){if($X[0]!="INDEX"){$X[2]=preg_replace('~ DESC$~','',$X[2]);$h=($X[2]=="DROP"?"\nDROP CONSTRAINT ".idf_escape($X[1]):"\nADD".($X[1]!=""?" CONSTRAINT ".idf_escape($X[1]):"")." $X[0] ".($X[0]=="PRIMARY"?"KEY ":"")."(".implode(", ",$X[2]).")");array_unshift($Kg,"ALTER TABLE ".table($R).$h);}elseif($X[2]=="DROP")$fc[]=idf_escape($X[1]);else$Kg[]="CREATE INDEX ".idf_escape($X[1]!=""?$X[1]:uniqid($R."_"))." ON ".table($R)." (".implode(", ",$X[2]).")";}if($fc)array_unshift($Kg,"DROP INDEX ".implode(", ",$fc));foreach($Kg
as$H){if(!queries($H))return
false;}return
true;}function
foreign_keys($R){$J=array();$H="SELECT c_list.CONSTRAINT_NAME as NAME,
c_src.COLUMN_NAME as SRC_COLUMN,
c_dest.OWNER as DEST_DB,
c_dest.TABLE_NAME as DEST_TABLE,
c_dest.COLUMN_NAME as DEST_COLUMN,
c_list.DELETE_RULE as ON_DELETE
FROM ALL_CONSTRAINTS c_list, ALL_CONS_COLUMNS c_src, ALL_CONS_COLUMNS c_dest
WHERE c_list.CONSTRAINT_NAME = c_src.CONSTRAINT_NAME
AND c_list.R_CONSTRAINT_NAME = c_dest.CONSTRAINT_NAME
AND c_list.CONSTRAINT_TYPE = 'R'
AND c_src.TABLE_NAME = ".q($R);foreach(get_rows($H)as$K)$J[$K['NAME']]=array("db"=>$K['DEST_DB'],"table"=>$K['DEST_TABLE'],"source"=>array($K['SRC_COLUMN']),"target"=>array($K['DEST_COLUMN']),"on_delete"=>$K['ON_DELETE'],"on_update"=>null,);return$J;}function
truncate_tables($T){return
apply_queries("TRUNCATE TABLE",$T);}function
drop_views($rj){return
apply_queries("DROP VIEW",$rj);}function
drop_tables($T){return
apply_queries("DROP TABLE",$T);}function
last_id($I){return
0;}function
schemas(){$J=get_vals("SELECT DISTINCT owner FROM dba_segments WHERE owner IN (SELECT username FROM dba_users WHERE default_tablespace NOT IN ('SYSTEM','SYSAUX')) ORDER BY 1");return($J?:get_vals("SELECT DISTINCT owner FROM all_tables WHERE tablespace_name = ".q(DB)." ORDER BY 1"));}function
get_schema(){return
get_val("SELECT sys_context('USERENV', 'SESSION_USER') FROM dual");}function
set_schema($oh,$g=null){if(!$g)$g=connection();return$g->query("ALTER SESSION SET CURRENT_SCHEMA = ".idf_escape($oh));}function
show_variables(){return
get_rows('SELECT name, display_value FROM v$parameter');}function
show_status(){$J=array();$L=get_rows('SELECT * FROM v$instance');foreach(reset($L)as$x=>$X)$J[]=array($x,$X);return$J;}function
process_list(){return
get_rows('SELECT
	sess.process AS "process",
	sess.username AS "user",
	sess.schemaname AS "schema",
	sess.status AS "status",
	sess.wait_class AS "wait_class",
	sess.seconds_in_wait AS "seconds_in_wait",
	sql.sql_text AS "sql_text",
	sess.machine AS "machine",
	sess.port AS "port"
FROM v$session sess LEFT OUTER JOIN v$sql sql
ON sql.sql_id = sess.sql_id
WHERE sess.type = \'USER\'
ORDER BY PROCESS
');}function
convert_field($m){}function
unconvert_field($m,$J){return$J;}function
support($Sc){return
preg_match('~^(columns|database|drop_col|indexes|descidx|processlist|scheme|sql|status|table|variables|view)$~',$Sc);}}add_driver("mssql","MS SQL");if(isset($_GET["mssql"])){define('Adminer\DRIVER',"mssql");if(extension_loaded("sqlsrv")&&$_GET["ext"]!="pdo"){class
Db
extends
SqlDb{var$extension="sqlsrv";private$link,$result;private
function
get_error(){$this->error="";foreach(sqlsrv_errors()as$l){$this->errno=$l["code"];$this->error
.="$l[message]\n";}$this->error=rtrim($this->error);}function
attach($N,$V,$F){$tb=array("UID"=>$V,"PWD"=>$F,"CharacterSet"=>"UTF-8");$Sh=adminer()->connectSsl();if(isset($Sh["Encrypt"]))$tb["Encrypt"]=$Sh["Encrypt"];if(isset($Sh["TrustServerCertificate"]))$tb["TrustServerCertificate"]=$Sh["TrustServerCertificate"];$j=adminer()->database();if($j!="")$tb["Database"]=$j;$this->link=@sqlsrv_connect(preg_replace('~:~',',',$N),$tb);if($this->link){$Zd=sqlsrv_server_info($this->link);$this->server_info=$Zd['SQLServerVersion'];}else$this->get_error();return($this->link?'':$this->error);}function
quote($Q){$Ui=strlen($Q)!=strlen(utf8_decode($Q));return($Ui?"N":"")."'".str_replace("'","''",$Q)."'";}function
select_db($Lb){return$this->query(use_sql($Lb));}function
query($H,$Ti=false){$I=sqlsrv_query($this->link,$H);$this->error="";if(!$I){$this->get_error();return
false;}return$this->store_result($I);}function
multi_query($H){$this->result=sqlsrv_query($this->link,$H);$this->error="";if(!$this->result){$this->get_error();return
false;}return
true;}function
store_result($I=null){if(!$I)$I=$this->result;if(!$I)return
false;if(sqlsrv_field_metadata($I))return
new
Result($I);$this->affected_rows=sqlsrv_rows_affected($I);return
true;}function
next_result(){return$this->result?!!sqlsrv_next_result($this->result):false;}}class
Result{var$num_rows;private$result,$offset=0,$fields;function
__construct($I){$this->result=$I;}private
function
convert($K){foreach((array)$K
as$x=>$X){if(is_a($X,'DateTime'))$K[$x]=$X->format("Y-m-d H:i:s");}return$K;}function
fetch_assoc(){return$this->convert(sqlsrv_fetch_array($this->result,SQLSRV_FETCH_ASSOC));}function
fetch_row(){return$this->convert(sqlsrv_fetch_array($this->result,SQLSRV_FETCH_NUMERIC));}function
fetch_field(){if(!$this->fields)$this->fields=sqlsrv_field_metadata($this->result);$m=$this->fields[$this->offset++];$J=new
\stdClass;$J->name=$m["Name"];$J->type=($m["Type"]==1?254:15);$J->charsetnr=0;return$J;}function
seek($D){for($s=0;$s<$D;$s++)sqlsrv_fetch($this->result);}function
__destruct(){sqlsrv_free_stmt($this->result);}}function
last_id($I){return
get_val("SELECT SCOPE_IDENTITY()");}function
explain($f,$H){$f->query("SET SHOWPLAN_ALL ON");$J=$f->query($H);$f->query("SET SHOWPLAN_ALL OFF");return$J;}}else{abstract
class
MssqlDb
extends
PdoDb{function
select_db($Lb){return$this->query(use_sql($Lb));}function
lastInsertId(){return$this->pdo->lastInsertId();}}function
last_id($I){return
connection()->lastInsertId();}function
explain($f,$H){}if(extension_loaded("pdo_sqlsrv")){class
Db
extends
MssqlDb{var$extension="PDO_SQLSRV";function
attach($N,$V,$F){return$this->dsn("sqlsrv:Server=".str_replace(":",",",$N),$V,$F);}}}elseif(extension_loaded("pdo_dblib")){class
Db
extends
MssqlDb{var$extension="PDO_DBLIB";function
attach($N,$V,$F){return$this->dsn("dblib:charset=utf8;host=".str_replace(":",";unix_socket=",preg_replace('~:(\d)~',';port=\1',$N)),$V,$F);}}}}class
Driver
extends
SqlDriver{static$Nc=array("SQLSRV","PDO_SQLSRV","PDO_DBLIB");static$pe="mssql";var$insertFunctions=array("date|time"=>"getdate");var$editFunctions=array("int|decimal|real|float|money|datetime"=>"+/-","char|text"=>"+",);var$operators=array("=","<",">","<=",">=","!=","LIKE","LIKE %%","IN","IS NULL","NOT LIKE","NOT IN","IS NOT NULL");var$functions=array("len","lower","round","upper");var$grouping=array("avg","count","count distinct","max","min","sum");var$generated=array("PERSISTED","VIRTUAL");var$onActions="NO ACTION|CASCADE|SET NULL|SET DEFAULT";static
function
connect($N,$V,$F){if($N=="")$N="localhost:1433";return
parent::connect($N,$V,$F);}function
__construct(Db$f){parent::__construct($f);$this->types=array('Numbers'=>array("tinyint"=>3,"smallint"=>5,"int"=>10,"bigint"=>20,"bit"=>1,"decimal"=>0,"real"=>12,"float"=>53,"smallmoney"=>10,"money"=>20),'Date and time'=>array("date"=>10,"smalldatetime"=>19,"datetime"=>19,"datetime2"=>19,"time"=>8,"datetimeoffset"=>10),'Strings'=>array("char"=>8000,"varchar"=>8000,"text"=>2147483647,"nchar"=>4000,"nvarchar"=>4000,"ntext"=>1073741823),'Binary'=>array("binary"=>8000,"varbinary"=>8000,"image"=>2147483647),);}function
insertUpdate($R,array$L,array$G){$n=fields($R);$bj=array();$Z=array();$O=reset($L);$e="c".implode(", c",range(1,count($O)));$Oa=0;$de=array();foreach($O
as$x=>$X){$Oa++;$C=idf_unescape($x);if(!$n[$C]["auto_increment"])$de[$x]="c$Oa";if(isset($G[$C]))$Z[]="$x = c$Oa";else$bj[]="$x = c$Oa";}$mj=array();foreach($L
as$O)$mj[]="(".implode(", ",$O).")";if($Z){$Od=queries("SET IDENTITY_INSERT ".table($R)." ON");$J=queries("MERGE ".table($R)." USING (VALUES\n\t".implode(",\n\t",$mj)."\n) AS source ($e) ON ".implode(" AND ",$Z).($bj?"\nWHEN MATCHED THEN UPDATE SET ".implode(", ",$bj):"")."\nWHEN NOT MATCHED THEN INSERT (".implode(", ",array_keys($Od?$O:$de)).") VALUES (".($Od?$e:implode(", ",$de)).");");if($Od)queries("SET IDENTITY_INSERT ".table($R)." OFF");}else$J=queries("INSERT INTO ".table($R)." (".implode(", ",array_keys($O)).") VALUES\n".implode(",\n",$mj));return$J;}function
begin(){return
queries("BEGIN TRANSACTION");}function
tableHelp($C,$ne=false){$Ge=array("sys"=>"catalog-views/sys-","INFORMATION_SCHEMA"=>"information-schema-views/",);$_=$Ge[get_schema()];if($_)return"relational-databases/system-$_".preg_replace('~_~','-',strtolower($C))."-transact-sql";}}function
idf_escape($u){return"[".str_replace("]","]]",$u)."]";}function
table($u){return($_GET["ns"]!=""?idf_escape($_GET["ns"]).".":"").idf_escape($u);}function
get_databases($ed){return
get_vals("SELECT name FROM sys.databases WHERE name NOT IN ('master', 'tempdb', 'model', 'msdb')");}function
limit($H,$Z,$z,$D=0,$xh=" "){return($z?" TOP (".($z+$D).")":"")." $H$Z";}function
limit1($R,$H,$Z,$xh="\n"){return
limit($H,$Z,1,0,$xh);}function
db_collation($j,$hb){return
get_val("SELECT collation_name FROM sys.databases WHERE name = ".q($j));}function
logged_user(){return
get_val("SELECT SUSER_NAME()");}function
tables_list(){return
get_key_vals("SELECT name, type_desc FROM sys.all_objects WHERE schema_id = SCHEMA_ID(".q(get_schema()).") AND type IN ('S', 'U', 'V') ORDER BY name");}function
count_tables($i){$J=array();foreach($i
as$j){connection()->select_db($j);$J[$j]=get_val("SELECT COUNT(*) FROM INFORMATION_SCHEMA.TABLES");}return$J;}function
table_status($C=""){$J=array();foreach(get_rows("SELECT ao.name AS Name, ao.type_desc AS Engine, (SELECT value FROM fn_listextendedproperty(default, 'SCHEMA', schema_name(schema_id), 'TABLE', ao.name, null, null)) AS Comment
FROM sys.all_objects AS ao
WHERE schema_id = SCHEMA_ID(".q(get_schema()).") AND type IN ('S', 'U', 'V') ".($C!=""?"AND name = ".q($C):"ORDER BY name"))as$K)$J[$K["Name"]]=$K;return$J;}function
is_view($S){return$S["Engine"]=="VIEW";}function
fk_support($S){return
true;}function
fields($R){$ob=get_key_vals("SELECT objname, cast(value as varchar(max)) FROM fn_listextendedproperty('MS_DESCRIPTION', 'schema', ".q(get_schema()).", 'table', ".q($R).", 'column', NULL)");$J=array();$ei=get_val("SELECT object_id FROM sys.all_objects WHERE schema_id = SCHEMA_ID(".q(get_schema()).") AND type IN ('S', 'U', 'V') AND name = ".q($R));foreach(get_rows("SELECT c.max_length, c.precision, c.scale, c.name, c.is_nullable, c.is_identity, c.collation_name, t.name type, d.definition [default], d.name default_constraint, i.is_primary_key
FROM sys.all_columns c
JOIN sys.types t ON c.user_type_id = t.user_type_id
LEFT JOIN sys.default_constraints d ON c.default_object_id = d.object_id
LEFT JOIN sys.index_columns ic ON c.object_id = ic.object_id AND c.column_id = ic.column_id
LEFT JOIN sys.indexes i ON ic.object_id = i.object_id AND ic.index_id = i.index_id
WHERE c.object_id = ".q($ei))as$K){$U=$K["type"];$y=(preg_match("~char|binary~",$U)?intval($K["max_length"])/($U[0]=='n'?2:1):($U=="decimal"?"$K[precision],$K[scale]":""));$J[$K["name"]]=array("field"=>$K["name"],"full_type"=>$U.($y?"($y)":""),"type"=>$U,"length"=>$y,"default"=>(preg_match("~^\('(.*)'\)$~",$K["default"],$B)?str_replace("''","'",$B[1]):$K["default"]),"default_constraint"=>$K["default_constraint"],"null"=>$K["is_nullable"],"auto_increment"=>$K["is_identity"],"collation"=>$K["collation_name"],"privileges"=>array("insert"=>1,"select"=>1,"update"=>1,"where"=>1,"order"=>1),"primary"=>$K["is_primary_key"],"comment"=>$ob[$K["name"]],);}foreach(get_rows("SELECT * FROM sys.computed_columns WHERE object_id = ".q($ei))as$K){$J[$K["name"]]["generated"]=($K["is_persisted"]?"PERSISTED":"VIRTUAL");$J[$K["name"]]["default"]=$K["definition"];}return$J;}function
indexes($R,$g=null){$J=array();foreach(get_rows("SELECT i.name, key_ordinal, is_unique, is_primary_key, c.name AS column_name, is_descending_key
FROM sys.indexes i
INNER JOIN sys.index_columns ic ON i.object_id = ic.object_id AND i.index_id = ic.index_id
INNER JOIN sys.columns c ON ic.object_id = c.object_id AND ic.column_id = c.column_id
WHERE OBJECT_NAME(i.object_id) = ".q($R),$g)as$K){$C=$K["name"];$J[$C]["type"]=($K["is_primary_key"]?"PRIMARY":($K["is_unique"]?"UNIQUE":"INDEX"));$J[$C]["lengths"]=array();$J[$C]["columns"][$K["key_ordinal"]]=$K["column_name"];$J[$C]["descs"][$K["key_ordinal"]]=($K["is_descending_key"]?'1':null);}return$J;}function
view($C){return
array("select"=>preg_replace('~^(?:[^[]|\[[^]]*])*\s+AS\s+~isU','',get_val("SELECT VIEW_DEFINITION FROM INFORMATION_SCHEMA.VIEWS WHERE TABLE_SCHEMA = SCHEMA_NAME() AND TABLE_NAME = ".q($C))));}function
collations(){$J=array();foreach(get_vals("SELECT name FROM fn_helpcollations()")as$c)$J[preg_replace('~_.*~','',$c)][]=$c;return$J;}function
information_schema($j){return
get_schema()=="INFORMATION_SCHEMA";}function
error(){return
nl_br(h(preg_replace('~^(\[[^]]*])+~m','',connection()->error)));}function
create_database($j,$c){return
queries("CREATE DATABASE ".idf_escape($j).(preg_match('~^[a-z0-9_]+$~i',$c)?" COLLATE $c":""));}function
drop_databases($i){return
queries("DROP DATABASE ".implode(", ",array_map('Adminer\idf_escape',$i)));}function
rename_database($C,$c){if(preg_match('~^[a-z0-9_]+$~i',$c))queries("ALTER DATABASE ".idf_escape(DB)." COLLATE $c");queries("ALTER DATABASE ".idf_escape(DB)." MODIFY NAME = ".idf_escape($C));return
true;}function
auto_increment(){return" IDENTITY".($_POST["Auto_increment"]!=""?"(".number($_POST["Auto_increment"]).",1)":"")." PRIMARY KEY";}function
alter_table($R,$C,$n,$gd,$mb,$vc,$c,$_a,$kg){$b=array();$ob=array();$Uf=fields($R);foreach($n
as$m){$d=idf_escape($m[0]);$X=$m[1];if(!$X)$b["DROP"][]=" COLUMN $d";else{$X[1]=preg_replace("~( COLLATE )'(\\w+)'~",'\1\2',$X[1]);$ob[$m[0]]=$X[5];unset($X[5]);if(preg_match('~ AS ~',$X[3]))unset($X[1],$X[2]);if($m[0]=="")$b["ADD"][]="\n  ".implode("",$X).($R==""?substr($gd[$X[0]],16+strlen($X[0])):"");else{$k=$X[3];unset($X[3]);unset($X[6]);if($d!=$X[0])queries("EXEC sp_rename ".q(table($R).".$d").", ".q(idf_unescape($X[0])).", 'COLUMN'");$b["ALTER COLUMN ".implode("",$X)][]="";$Tf=$Uf[$m[0]];if(default_value($Tf)!=$k){if($Tf["default"]!==null)$b["DROP"][]=" ".idf_escape($Tf["default_constraint"]);if($k)$b["ADD"][]="\n $k FOR $d";}}}}if($R=="")return
queries("CREATE TABLE ".table($C)." (".implode(",",(array)$b["ADD"])."\n)");if($R!=$C)queries("EXEC sp_rename ".q(table($R)).", ".q($C));if($gd)$b[""]=$gd;foreach($b
as$x=>$X){if(!queries("ALTER TABLE ".table($C)." $x".implode(",",$X)))return
false;}foreach($ob
as$x=>$X){$mb=substr($X,9);queries("EXEC sp_dropextendedproperty @name = N'MS_Description', @level0type = N'Schema', @level0name = ".q(get_schema()).", @level1type = N'Table', @level1name = ".q($C).", @level2type = N'Column', @level2name = ".q($x));queries("EXEC sp_addextendedproperty
@name = N'MS_Description',
@value = $mb,
@level0type = N'Schema',
@level0name = ".q(get_schema()).",
@level1type = N'Table',
@level1name = ".q($C).",
@level2type = N'Column',
@level2name = ".q($x));}return
true;}function
alter_indexes($R,$b){$v=array();$fc=array();foreach($b
as$X){if($X[2]=="DROP"){if($X[0]=="PRIMARY")$fc[]=idf_escape($X[1]);else$v[]=idf_escape($X[1])." ON ".table($R);}elseif(!queries(($X[0]!="PRIMARY"?"CREATE $X[0] ".($X[0]!="INDEX"?"INDEX ":"").idf_escape($X[1]!=""?$X[1]:uniqid($R."_"))." ON ".table($R):"ALTER TABLE ".table($R)." ADD PRIMARY KEY")." (".implode(", ",$X[2]).")"))return
false;}return(!$v||queries("DROP INDEX ".implode(", ",$v)))&&(!$fc||queries("ALTER TABLE ".table($R)." DROP ".implode(", ",$fc)));}function
found_rows($S,$Z){}function
foreign_keys($R){$J=array();$Ef=array("CASCADE","NO ACTION","SET NULL","SET DEFAULT");foreach(get_rows("EXEC sp_fkeys @fktable_name = ".q($R).", @fktable_owner = ".q(get_schema()))as$K){$p=&$J[$K["FK_NAME"]];$p["db"]=$K["PKTABLE_QUALIFIER"];$p["ns"]=$K["PKTABLE_OWNER"];$p["table"]=$K["PKTABLE_NAME"];$p["on_update"]=$Ef[$K["UPDATE_RULE"]];$p["on_delete"]=$Ef[$K["DELETE_RULE"]];$p["source"][]=$K["FKCOLUMN_NAME"];$p["target"][]=$K["PKCOLUMN_NAME"];}return$J;}function
truncate_tables($T){return
apply_queries("TRUNCATE TABLE",$T);}function
drop_views($rj){return
queries("DROP VIEW ".implode(", ",array_map('Adminer\table',$rj)));}function
drop_tables($T){return
queries("DROP TABLE ".implode(", ",array_map('Adminer\table',$T)));}function
move_tables($T,$rj,$oi){return
apply_queries("ALTER SCHEMA ".idf_escape($oi)." TRANSFER",array_merge($T,$rj));}function
trigger($C,$R){if($C=="")return
array();$L=get_rows("SELECT s.name [Trigger],
CASE WHEN OBJECTPROPERTY(s.id, 'ExecIsInsertTrigger') = 1 THEN 'INSERT' WHEN OBJECTPROPERTY(s.id, 'ExecIsUpdateTrigger') = 1 THEN 'UPDATE' WHEN OBJECTPROPERTY(s.id, 'ExecIsDeleteTrigger') = 1 THEN 'DELETE' END [Event],
CASE WHEN OBJECTPROPERTY(s.id, 'ExecIsInsteadOfTrigger') = 1 THEN 'INSTEAD OF' ELSE 'AFTER' END [Timing],
c.text
FROM sysobjects s
JOIN syscomments c ON s.id = c.id
WHERE s.xtype = 'TR' AND s.name = ".q($C));$J=reset($L);if($J)$J["Statement"]=preg_replace('~^.+\s+AS\s+~isU','',$J["text"]);return$J;}function
triggers($R){$J=array();foreach(get_rows("SELECT sys1.name,
CASE WHEN OBJECTPROPERTY(sys1.id, 'ExecIsInsertTrigger') = 1 THEN 'INSERT' WHEN OBJECTPROPERTY(sys1.id, 'ExecIsUpdateTrigger') = 1 THEN 'UPDATE' WHEN OBJECTPROPERTY(sys1.id, 'ExecIsDeleteTrigger') = 1 THEN 'DELETE' END [Event],
CASE WHEN OBJECTPROPERTY(sys1.id, 'ExecIsInsteadOfTrigger') = 1 THEN 'INSTEAD OF' ELSE 'AFTER' END [Timing]
FROM sysobjects sys1
JOIN sysobjects sys2 ON sys1.parent_obj = sys2.id
WHERE sys1.xtype = 'TR' AND sys2.name = ".q($R))as$K)$J[$K["name"]]=array($K["Timing"],$K["Event"]);return$J;}function
trigger_options(){return
array("Timing"=>array("AFTER","INSTEAD OF"),"Event"=>array("INSERT","UPDATE","DELETE"),"Type"=>array("AS"),);}function
schemas(){return
get_vals("SELECT name FROM sys.schemas");}function
get_schema(){if($_GET["ns"]!="")return$_GET["ns"];return
get_val("SELECT SCHEMA_NAME()");}function
set_schema($mh){$_GET["ns"]=$mh;return
true;}function
create_sql($R,$_a,$Xh){if(is_view(table_status1($R))){$qj=view($R);return"CREATE VIEW ".table($R)." AS $qj[select]";}$n=array();$G=false;foreach(fields($R)as$C=>$m){$X=process_field($m,$m);if($X[6])$G=true;$n[]=implode("",$X);}foreach(indexes($R)as$C=>$v){if(!$G||$v["type"]!="PRIMARY"){$e=array();foreach($v["columns"]as$x=>$X)$e[]=idf_escape($X).($v["descs"][$x]?" DESC":"");$C=idf_escape($C);$n[]=($v["type"]=="INDEX"?"INDEX $C":"CONSTRAINT $C ".($v["type"]=="UNIQUE"?"UNIQUE":"PRIMARY KEY"))." (".implode(", ",$e).")";}}foreach(driver()->checkConstraints($R)as$C=>$Va)$n[]="CONSTRAINT ".idf_escape($C)." CHECK ($Va)";return"CREATE TABLE ".table($R)." (\n\t".implode(",\n\t",$n)."\n)";}function
foreign_keys_sql($R){$n=array();foreach(foreign_keys($R)as$gd)$n[]=ltrim(format_foreign_key($gd));return($n?"ALTER TABLE ".table($R)." ADD\n\t".implode(",\n\t",$n).";\n\n":"");}function
truncate_sql($R){return"TRUNCATE TABLE ".table($R);}function
use_sql($Lb){return"USE ".idf_escape($Lb);}function
trigger_sql($R){$J="";foreach(triggers($R)as$C=>$Mi)$J
.=create_trigger(" ON ".table($R),trigger($C,$R)).";";return$J;}function
convert_field($m){}function
unconvert_field($m,$J){return$J;}function
support($Sc){return
preg_match('~^(check|comment|columns|database|drop_col|dump|indexes|descidx|scheme|sql|table|trigger|view|view_trigger)$~',$Sc);}}class
Adminer{static$fe;var$error='';function
name(){return"<a href='https://www.adminer.org/'".target_blank()." id='h1'><img src='".h(preg_replace("~\\?.*~","",ME)."?file=logo.png&version=5.2.1")."' width='24' height='24' alt='' id='logo'>Adminer</a>";}function
credentials(){return
array(SERVER,$_GET["username"],get_password());}function
connectSsl(){}function
permanentLogin($h=false){return
password_file($h);}function
bruteForceKey(){return$_SERVER["REMOTE_ADDR"];}function
serverName($N){return
h($N);}function
database(){return
DB;}function
databases($ed=true){return
get_databases($ed);}function
pluginsLinks(){}function
operators(){return
driver()->operators;}function
schemas(){return
schemas();}function
queryTimeout(){return
2;}function
headers(){}function
csp(array$Eb){return$Eb;}function
head($Ib=null){return
true;}function
css(){$J=array();foreach(array("","-dark")as$ff){$o="adminer$ff.css";if(file_exists($o))$J[]="$o?v=".crc32(file_get_contents($o));}return$J;}function
loginForm(){echo"<table class='layout'>\n",adminer()->loginFormField('driver','<tr><th>'.'System'.'<td>',html_select("auth[driver]",SqlDriver::$ec,DRIVER,"loginDriver(this);")),adminer()->loginFormField('server','<tr><th>'.'Server'.'<td>','<input name="auth[server]" value="'.h(SERVER).'" title="hostname[:port]" placeholder="localhost" autocapitalize="off">'),adminer()->loginFormField('username','<tr><th>'.'Username'.'<td>','<input name="auth[username]" id="username" autofocus value="'.h($_GET["username"]).'" autocomplete="username" autocapitalize="off">'.script("const authDriver = qs('#username').form['auth[driver]']; authDriver && authDriver.onchange();")),adminer()->loginFormField('password','<tr><th>'.'Password'.'<td>','<input type="password" name="auth[password]" autocomplete="current-password">'),adminer()->loginFormField('db','<tr><th>'.'Database'.'<td>','<input name="auth[db]" value="'.h($_GET["db"]).'" autocapitalize="off">'),"</table>\n","<p><input type='submit' value='".'Login'."'>\n",checkbox("auth[permanent]",1,$_COOKIE["adminer_permanent"],'Permanent login')."\n";}function
loginFormField($C,$Ed,$Y){return$Ed.$Y."\n";}function
login($He,$F){if($F=="")return
sprintf('Adminer does not support accessing a database without a password, <a href="https://www.adminer.org/en/password/"%s>more information</a>.',target_blank());return
true;}function
tableName(array$di){return
h($di["Name"]);}function
fieldName(array$m,$Nf=0){$U=$m["full_type"];$mb=$m["comment"];return'<span title="'.h($U.($mb!=""?($U?": ":"").$mb:'')).'">'.h($m["field"]).'</span>';}function
selectLinks(array$di,$O=""){echo'<p class="links">';$Ge=array("select"=>'Select data');if(support("table")||support("indexes"))$Ge["table"]='Show structure';$ne=false;if(support("table")){$ne=is_view($di);if($ne)$Ge["view"]='Alter view';else$Ge["create"]='Alter table';}if($O!==null)$Ge["edit"]='New item';$C=$di["Name"];foreach($Ge
as$x=>$X)echo" <a href='".h(ME)."$x=".urlencode($C).($x=="edit"?$O:"")."'".bold(isset($_GET[$x])).">$X</a>";echo
doc_link(array(JUSH=>driver()->tableHelp($C,$ne)),"?"),"\n";}function
foreignKeys($R){return
foreign_keys($R);}function
backwardKeys($R,$ci){return
array();}function
backwardKeysPrint(array$Da,array$K){}function
selectQuery($H,$Th,$Qc=false){$J="</p>\n";if(!$Qc&&($uj=driver()->warnings())){$t="warnings";$J=", <a href='#$t'>".'Warnings'."</a>".script("qsl('a').onclick = partial(toggle, '$t');","")."$J<div id='$t' class='hidden'>\n$uj</div>\n";}return"<p><code class='jush-".JUSH."'>".h(str_replace("\n"," ",$H))."</code> <span class='time'>(".format_time($Th).")</span>".(support("sql")?" <a href='".h(ME)."sql=".urlencode($H)."'>".'Edit'."</a>":"").$J;}function
sqlCommandQuery($H){return
shorten_utf8(trim($H),1000);}function
sqlPrintAfter(){}function
rowDescription($R){return"";}function
rowDescriptions(array$L,array$hd){return$L;}function
selectLink($X,array$m){}function
selectVal($X,$_,array$m,$Xf){$J=($X===null?"<i>NULL</i>":(preg_match("~char|binary|boolean~",$m["type"])&&!preg_match("~var~",$m["type"])?"<code>$X</code>":(preg_match('~json~',$m["type"])?"<code class='jush-js'>$X</code>":$X)));if(preg_match('~blob|bytea|raw|file~',$m["type"])&&!is_utf8($X))$J="<i>".lang_format(array('%d byte','%d bytes'),strlen($Xf))."</i>";return($_?"<a href='".h($_)."'".(is_url($_)?target_blank():"").">$J</a>":$J);}function
editVal($X,array$m){return$X;}function
config(){return
array();}function
tableStructurePrint(array$n,$di=null){echo"<div class='scrollable'>\n","<table class='nowrap odds'>\n","<thead><tr><th>".'Column'."<td>".'Type'.(support("comment")?"<td>".'Comment':"")."</thead>\n";$Wh=driver()->structuredTypes();foreach($n
as$m){echo"<tr><th>".h($m["field"]);$U=h($m["full_type"]);$c=h($m["collation"]);echo"<td><span title='$c'>".(in_array($U,(array)$Wh['User types'])?"<a href='".h(ME.'type='.urlencode($U))."'>$U</a>":$U.($c&&isset($di["Collation"])&&$c!=$di["Collation"]?" $c":""))."</span>",($m["null"]?" <i>NULL</i>":""),($m["auto_increment"]?" <i>".'Auto Increment'."</i>":"");$k=h($m["default"]);echo(isset($m["default"])?" <span title='".'Default value'."'>[<b>".($m["generated"]?"<code class='jush-".JUSH."'>$k</code>":$k)."</b>]</span>":""),(support("comment")?"<td>".h($m["comment"]):""),"\n";}echo"</table>\n","</div>\n";}function
tableIndexesPrint(array$w){echo"<table>\n";foreach($w
as$C=>$v){ksort($v["columns"]);$Cg=array();foreach($v["columns"]as$x=>$X)$Cg[]="<i>".h($X)."</i>".($v["lengths"][$x]?"(".$v["lengths"][$x].")":"").($v["descs"][$x]?" DESC":"");echo"<tr title='".h($C)."'><th>$v[type]<td>".implode(", ",$Cg)."\n";}echo"</table>\n";}function
selectColumnsPrint(array$M,array$e){print_fieldset("select",'Select',$M);$s=0;$M[""]=array();foreach($M
as$x=>$X){$X=idx($_GET["columns"],$x,array());$d=select_input(" name='columns[$s][col]'",$e,$X["col"],($x!==""?"selectFieldChange":"selectAddRow"));echo"<div>".(driver()->functions||driver()->grouping?html_select("columns[$s][fun]",array(-1=>"")+array_filter(array('Functions'=>driver()->functions,'Aggregation'=>driver()->grouping)),$X["fun"]).on_help("event.target.value && event.target.value.replace(/ |\$/, '(') + ')'",1).script("qsl('select').onchange = function () { helpClose();".($x!==""?"":" qsl('select, input', this.parentNode).onchange();")." };","")."($d)":$d)."</div>\n";$s++;}echo"</div></fieldset>\n";}function
selectSearchPrint(array$Z,array$e,array$w){print_fieldset("search",'Search',$Z);foreach($w
as$s=>$v){if($v["type"]=="FULLTEXT")echo"<div>(<i>".implode("</i>, <i>",array_map('Adminer\h',$v["columns"]))."</i>) AGAINST"," <input type='search' name='fulltext[$s]' value='".h($_GET["fulltext"][$s])."'>",script("qsl('input').oninput = selectFieldChange;",""),checkbox("boolean[$s]",1,isset($_GET["boolean"][$s]),"BOOL"),"</div>\n";}$Sa="this.parentNode.firstChild.onchange();";foreach(array_merge((array)$_GET["where"],array(array()))as$s=>$X){if(!$X||("$X[col]$X[val]"!=""&&in_array($X["op"],adminer()->operators())))echo"<div>".select_input(" name='where[$s][col]'",$e,$X["col"],($X?"selectFieldChange":"selectAddRow"),"(".'anywhere'.")"),html_select("where[$s][op]",adminer()->operators(),$X["op"],$Sa),"<input type='search' name='where[$s][val]' value='".h($X["val"])."'>",script("mixin(qsl('input'), {oninput: function () { $Sa }, onkeydown: selectSearchKeydown, onsearch: selectSearchSearch});",""),"</div>\n";}echo"</div></fieldset>\n";}function
selectOrderPrint(array$Nf,array$e,array$w){print_fieldset("sort",'Sort',$Nf);$s=0;foreach((array)$_GET["order"]as$x=>$X){if($X!=""){echo"<div>".select_input(" name='order[$s]'",$e,$X,"selectFieldChange"),checkbox("desc[$s]",1,isset($_GET["desc"][$x]),'descending')."</div>\n";$s++;}}echo"<div>".select_input(" name='order[$s]'",$e,"","selectAddRow"),checkbox("desc[$s]",1,false,'descending')."</div>\n","</div></fieldset>\n";}function
selectLimitPrint($z){echo"<fieldset><legend>".'Limit'."</legend><div>","<input type='number' name='limit' class='size' value='".intval($z)."'>",script("qsl('input').oninput = selectFieldChange;",""),"</div></fieldset>\n";}function
selectLengthPrint($ui){if($ui!==null)echo"<fieldset><legend>".'Text length'."</legend><div>","<input type='number' name='text_length' class='size' value='".h($ui)."'>","</div></fieldset>\n";}function
selectActionPrint(array$w){echo"<fieldset><legend>".'Action'."</legend><div>","<input type='submit' value='".'Select'."'>"," <span id='noindex' title='".'Full table scan'."'></span>","<script".nonce().">\n","const indexColumns = ";$e=array();foreach($w
as$v){$Hb=reset($v["columns"]);if($v["type"]!="FULLTEXT"&&$Hb)$e[$Hb]=1;}$e[""]=1;foreach($e
as$x=>$X)json_row($x);echo";\n","selectFieldChange.call(qs('#form')['select']);\n","</script>\n","</div></fieldset>\n";}function
selectCommandPrint(){return!information_schema(DB);}function
selectImportPrint(){return!information_schema(DB);}function
selectEmailPrint(array$sc,array$e){}function
selectColumnsProcess(array$e,array$w){$M=array();$td=array();foreach((array)$_GET["columns"]as$x=>$X){if($X["fun"]=="count"||($X["col"]!=""&&(!$X["fun"]||in_array($X["fun"],driver()->functions)||in_array($X["fun"],driver()->grouping)))){$M[$x]=apply_sql_function($X["fun"],($X["col"]!=""?idf_escape($X["col"]):"*"));if(!in_array($X["fun"],driver()->grouping))$td[]=$M[$x];}}return
array($M,$td);}function
selectSearchProcess(array$n,array$w){$J=array();foreach($w
as$s=>$v){if($v["type"]=="FULLTEXT"&&$_GET["fulltext"][$s]!="")$J[]="MATCH (".implode(", ",array_map('Adminer\idf_escape',$v["columns"])).") AGAINST (".q($_GET["fulltext"][$s]).(isset($_GET["boolean"][$s])?" IN BOOLEAN MODE":"").")";}foreach((array)$_GET["where"]as$x=>$X){$fb=$X["col"];if("$fb$X[val]"!=""&&in_array($X["op"],adminer()->operators())){$qb=array();foreach(($fb!=""?array($fb=>$n[$fb]):$n)as$C=>$m){$_g="";$pb=" $X[op]";if(preg_match('~IN$~',$X["op"])){$Sd=process_length($X["val"]);$pb
.=" ".($Sd!=""?$Sd:"(NULL)");}elseif($X["op"]=="SQL")$pb=" $X[val]";elseif(preg_match('~^(I?LIKE) %%$~',$X["op"],$B))$pb=" $B[1] ".adminer()->processInput($m,"%$X[val]%");elseif($X["op"]=="FIND_IN_SET"){$_g="$X[op](".q($X["val"]).", ";$pb=")";}elseif(!preg_match('~NULL$~',$X["op"]))$pb
.=" ".adminer()->processInput($m,$X["val"]);if($fb!=""||(isset($m["privileges"]["where"])&&(preg_match('~^[-\d.'.(preg_match('~IN$~',$X["op"])?',':'').']+$~',$X["val"])||!preg_match('~'.number_type().'|bit~',$m["type"]))&&(!preg_match("~[\x80-\xFF]~",$X["val"])||preg_match('~char|text|enum|set~',$m["type"]))&&(!preg_match('~date|timestamp~',$m["type"])||preg_match('~^\d+-\d+-\d+~',$X["val"]))))$qb[]=$_g.driver()->convertSearch(idf_escape($C),$X,$m).$pb;}$J[]=(count($qb)==1?$qb[0]:($qb?"(".implode(" OR ",$qb).")":"1 = 0"));}}return$J;}function
selectOrderProcess(array$n,array$w){$J=array();foreach((array)$_GET["order"]as$x=>$X){if($X!="")$J[]=(preg_match('~^((COUNT\(DISTINCT |[A-Z0-9_]+\()(`(?:[^`]|``)+`|"(?:[^"]|"")+")\)|COUNT\(\*\))$~',$X)?$X:idf_escape($X)).(isset($_GET["desc"][$x])?" DESC":"");}return$J;}function
selectLimitProcess(){return(isset($_GET["limit"])?intval($_GET["limit"]):50);}function
selectLengthProcess(){return(isset($_GET["text_length"])?"$_GET[text_length]":"100");}function
selectEmailProcess(array$Z,array$hd){return
false;}function
selectQueryBuild(array$M,array$Z,array$td,array$Nf,$z,$E){return"";}function
messageQuery($H,$vi,$Qc=false){restart_session();$Gd=&get_session("queries");if(!idx($Gd,$_GET["db"]))$Gd[$_GET["db"]]=array();if(strlen($H)>1e6)$H=preg_replace('~[\x80-\xFF]+$~','',substr($H,0,1e6))."\n…";$Gd[$_GET["db"]][]=array($H,time(),$vi);$Ph="sql-".count($Gd[$_GET["db"]]);$J="<a href='#$Ph' class='toggle'>".'SQL command'."</a>\n";if(!$Qc&&($uj=driver()->warnings())){$t="warnings-".count($Gd[$_GET["db"]]);$J="<a href='#$t' class='toggle'>".'Warnings'."</a>, $J<div id='$t' class='hidden'>\n$uj</div>\n";}return" <span class='time'>".@date("H:i:s")."</span>"." $J<div id='$Ph' class='hidden'><pre><code class='jush-".JUSH."'>".shorten_utf8($H,1000)."</code></pre>".($vi?" <span class='time'>($vi)</span>":'').(support("sql")?'<p><a href="'.h(str_replace("db=".urlencode(DB),"db=".urlencode($_GET["db"]),ME).'sql=&history='.(count($Gd[$_GET["db"]])-1)).'">'.'Edit'.'</a>':'').'</div>';}function
editRowPrint($R,array$n,$K,$bj){}function
editFunctions(array$m){$J=($m["null"]?"NULL/":"");$bj=isset($_GET["select"])||where($_GET);foreach(array(driver()->insertFunctions,driver()->editFunctions)as$x=>$od){if(!$x||(!isset($_GET["call"])&&$bj)){foreach($od
as$og=>$X){if(!$og||preg_match("~$og~",$m["type"]))$J
.="/$X";}}if($x&&$od&&!preg_match('~set|blob|bytea|raw|file|bool~',$m["type"]))$J
.="/SQL";}if($m["auto_increment"]&&!$bj)$J='Auto Increment';return
explode("/",$J);}function
editInput($R,array$m,$ya,$Y){if($m["type"]=="enum")return(isset($_GET["select"])?"<label><input type='radio'$ya value='-1' checked><i>".'original'."</i></label> ":"").($m["null"]?"<label><input type='radio'$ya value=''".($Y!==null||isset($_GET["select"])?"":" checked")."><i>NULL</i></label> ":"").enum_input("radio",$ya,$m,$Y,$Y===0?0:null);return"";}function
editHint($R,array$m,$Y){return"";}function
processInput(array$m,$Y,$r=""){if($r=="SQL")return$Y;$C=$m["field"];$J=q($Y);if(preg_match('~^(now|getdate|uuid)$~',$r))$J="$r()";elseif(preg_match('~^current_(date|timestamp)$~',$r))$J=$r;elseif(preg_match('~^([+-]|\|\|)$~',$r))$J=idf_escape($C)." $r $J";elseif(preg_match('~^[+-] interval$~',$r))$J=idf_escape($C)." $r ".(preg_match("~^(\\d+|'[0-9.: -]') [A-Z_]+\$~i",$Y)?$Y:$J);elseif(preg_match('~^(addtime|subtime|concat)$~',$r))$J="$r(".idf_escape($C).", $J)";elseif(preg_match('~^(md5|sha1|password|encrypt)$~',$r))$J="$r($J)";return
unconvert_field($m,$J);}function
dumpOutput(){$J=array('text'=>'open','file'=>'save');if(function_exists('gzencode'))$J['gz']='gzip';return$J;}function
dumpFormat(){return(support("dump")?array('sql'=>'SQL'):array())+array('csv'=>'CSV,','csv;'=>'CSV;','tsv'=>'TSV');}function
dumpDatabase($j){}function
dumpTable($R,$Xh,$ne=0){if($_POST["format"]!="sql"){echo"\xef\xbb\xbf";if($Xh)dump_csv(array_keys(fields($R)));}else{if($ne==2){$n=array();foreach(fields($R)as$C=>$m)$n[]=idf_escape($C)." $m[full_type]";$h="CREATE TABLE ".table($R)." (".implode(", ",$n).")";}else$h=create_sql($R,$_POST["auto_increment"],$Xh);set_utf8mb4($h);if($Xh&&$h){if($Xh=="DROP+CREATE"||$ne==1)echo"DROP ".($ne==2?"VIEW":"TABLE")." IF EXISTS ".table($R).";\n";if($ne==1)$h=remove_definer($h);echo"$h;\n\n";}}}function
dumpData($R,$Xh,$H){if($Xh){$Pe=(JUSH=="sqlite"?0:1048576);$n=array();$Pd=false;if($_POST["format"]=="sql"){if($Xh=="TRUNCATE+INSERT")echo
truncate_sql($R).";\n";$n=fields($R);if(JUSH=="mssql"){foreach($n
as$m){if($m["auto_increment"]){echo"SET IDENTITY_INSERT ".table($R)." ON;\n";$Pd=true;break;}}}}$I=connection()->query($H,1);if($I){$de="";$Na="";$se=array();$pd=array();$Zh="";$Tc=($R!=''?'fetch_assoc':'fetch_row');$Ab=0;while($K=$I->$Tc()){if(!$se){$mj=array();foreach($K
as$X){$m=$I->fetch_field();if(idx($n[$m->name],'generated')){$pd[$m->name]=true;continue;}$se[]=$m->name;$x=idf_escape($m->name);$mj[]="$x = VALUES($x)";}$Zh=($Xh=="INSERT+UPDATE"?"\nON DUPLICATE KEY UPDATE ".implode(", ",$mj):"").";\n";}if($_POST["format"]!="sql"){if($Xh=="table"){dump_csv($se);$Xh="INSERT";}dump_csv($K);}else{if(!$de)$de="INSERT INTO ".table($R)." (".implode(", ",array_map('Adminer\idf_escape',$se)).") VALUES";foreach($K
as$x=>$X){if($pd[$x]){unset($K[$x]);continue;}$m=$n[$x];$K[$x]=($X!==null?unconvert_field($m,preg_match(number_type(),$m["type"])&&!preg_match('~\[~',$m["full_type"])&&is_numeric($X)?$X:q(($X===false?0:$X))):"NULL");}$kh=($Pe?"\n":" ")."(".implode(",\t",$K).")";if(!$Na)$Na=$de.$kh;elseif(JUSH=='mssql'?$Ab%1000!=0:strlen($Na)+4+strlen($kh)+strlen($Zh)<$Pe)$Na
.=",$kh";else{echo$Na.$Zh;$Na=$de.$kh;}}$Ab++;}if($Na)echo$Na.$Zh;}elseif($_POST["format"]=="sql")echo"-- ".str_replace("\n"," ",connection()->error)."\n";if($Pd)echo"SET IDENTITY_INSERT ".table($R)." OFF;\n";}}function
dumpFilename($Nd){return
friendly_url($Nd!=""?$Nd:(SERVER!=""?SERVER:"localhost"));}function
dumpHeaders($Nd,$hf=false){$ag=$_POST["output"];$Lc=(preg_match('~sql~',$_POST["format"])?"sql":($hf?"tar":"csv"));header("Content-Type: ".($ag=="gz"?"application/x-gzip":($Lc=="tar"?"application/x-tar":($Lc=="sql"||$ag!="file"?"text/plain":"text/csv")."; charset=utf-8")));if($ag=="gz"){ob_start(function($Q){return
gzencode($Q);},1e6);}return$Lc;}function
dumpFooter(){if($_POST["format"]=="sql")echo"-- ".gmdate("Y-m-d H:i:s e")."\n";}function
importServerPath(){return"adminer.sql";}function
homepage(){echo'<p class="links">'.($_GET["ns"]==""&&support("database")?'<a href="'.h(ME).'database=">'.'Alter database'."</a>\n":""),(support("scheme")?"<a href='".h(ME)."scheme='>".($_GET["ns"]!=""?'Alter schema':'Create schema')."</a>\n":""),($_GET["ns"]!==""?'<a href="'.h(ME).'schema=">'.'Database schema'."</a>\n":""),(support("privileges")?"<a href='".h(ME)."privileges='>".'Privileges'."</a>\n":"");return
true;}function
navigation($ef){echo"<h1>".adminer()->name()." <span class='version'>".VERSION;$pf=$_COOKIE["adminer_version"];echo" <a href='https://www.adminer.org/#download'".target_blank()." id='version'>".(version_compare(VERSION,$pf)<0?h($pf):"")."</a>","</span></h1>\n";if($ef=="auth"){$ag="";foreach((array)$_SESSION["pwds"]as$oj=>$Bh){foreach($Bh
as$N=>$jj){$C=h(get_setting("vendor-$oj-$N")?:get_driver($oj));foreach($jj
as$V=>$F){if($F!==null){$Ob=$_SESSION["db"][$oj][$N][$V];foreach(($Ob?array_keys($Ob):array(""))as$j)$ag
.="<li><a href='".h(auth_url($oj,$N,$V,$j))."'>($C) ".h($V.($N!=""?"@".adminer()->serverName($N):"").($j!=""?" - $j":""))."</a>\n";}}}}if($ag)echo"<ul id='logins'>\n$ag</ul>\n".script("mixin(qs('#logins'), {onmouseover: menuOver, onmouseout: menuOut});");}else{$T=array();if($_GET["ns"]!==""&&!$ef&&DB!=""){connection()->select_db(DB);$T=table_status('',true);}adminer()->syntaxHighlighting($T);adminer()->databasesPrint($ef);$ia=array();if(DB==""||!$ef){if(support("sql")){$ia[]="<a href='".h(ME)."sql='".bold(isset($_GET["sql"])&&!isset($_GET["import"])).">".'SQL command'."</a>";$ia[]="<a href='".h(ME)."import='".bold(isset($_GET["import"])).">".'Import'."</a>";}$ia[]="<a href='".h(ME)."dump=".urlencode(isset($_GET["table"])?$_GET["table"]:$_GET["select"])."' id='dump'".bold(isset($_GET["dump"])).">".'Export'."</a>";}$Td=$_GET["ns"]!==""&&!$ef&&DB!="";if($Td)$ia[]='<a href="'.h(ME).'create="'.bold($_GET["create"]==="").">".'Create table'."</a>";echo($ia?"<p class='links'>\n".implode("\n",$ia)."\n":"");if($Td){if($T)adminer()->tablesPrint($T);else
echo"<p class='message'>".'No tables.'."</p>\n";}}}function
syntaxHighlighting(array$T){echo
script_src(preg_replace("~\\?.*~","",ME)."?file=jush.js&version=5.2.1",true);if(support("sql")){echo"<script".nonce().">\n";if($T){$Ge=array();foreach($T
as$R=>$U)$Ge[]=preg_quote($R,'/');echo"var jushLinks = { ".JUSH.": [ '".js_escape(ME).(support("table")?"table=":"select=")."\$&', /\\b(".implode("|",$Ge).")\\b/g ] };\n";foreach(array("bac","bra","sqlite_quo","mssql_bra")as$X)echo"jushLinks.$X = jushLinks.".JUSH.";\n";if(isset($_GET["sql"])||isset($_GET["trigger"])||isset($_GET["check"])){$ki=array_fill_keys(array_keys($T),array());foreach(driver()->allFields()as$R=>$n){foreach($n
as$m)$ki[$R][]=$m["field"];}echo"addEventListener('DOMContentLoaded', () => { autocompleter = jush.autocompleteSql('".idf_escape("")."', ".json_encode($ki)."); });\n";}}echo"</script>\n";}echo
script("syntaxHighlighting('".preg_replace('~^(\d\.?\d).*~s','\1',connection()->server_info)."', '".connection()->flavor."');");}function
databasesPrint($ef){$i=adminer()->databases();if(DB&&$i&&!in_array(DB,$i))array_unshift($i,DB);echo"<form action=''>\n<p id='dbs'>\n";hidden_fields_get();$Mb=script("mixin(qsl('select'), {onmousedown: dbMouseDown, onchange: dbChange});");echo"<label title='".'Database'."'>".'DB'.": ".($i?html_select("db",array(""=>"")+$i,DB).$Mb:"<input name='db' value='".h(DB)."' autocapitalize='off' size='19'>\n")."</label>","<input type='submit' value='".'Use'."'".($i?" class='hidden'":"").">\n";if(support("scheme")){if($ef!="db"&&DB!=""&&connection()->select_db(DB)){echo"<br><label>".'Schema'.": ".html_select("ns",array(""=>"")+adminer()->schemas(),$_GET["ns"])."$Mb</label>";if($_GET["ns"]!="")set_schema($_GET["ns"]);}}foreach(array("import","sql","schema","dump","privileges")as$X){if(isset($_GET[$X])){echo
input_hidden($X);break;}}echo"</p></form>\n";}function
tablesPrint(array$T){echo"<ul id='tables'>".script("mixin(qs('#tables'), {onmouseover: menuOver, onmouseout: menuOut});");foreach($T
as$R=>$P){$R="$R";$C=adminer()->tableName($P);if($C!="")echo'<li><a href="'.h(ME).'select='.urlencode($R).'"'.bold($_GET["select"]==$R||$_GET["edit"]==$R,"select")." title='".'Select data'."'>".'select'."</a> ",(support("table")||support("indexes")?'<a href="'.h(ME).'table='.urlencode($R).'"'.bold(in_array($R,array($_GET["table"],$_GET["create"],$_GET["indexes"],$_GET["foreign"],$_GET["trigger"],$_GET["check"],$_GET["view"])),(is_view($P)?"view":"structure"))." title='".'Show structure'."'>$C</a>":"<span>$C</span>")."\n";}echo"</ul>\n";}}class
Plugins{private
static$ta=array('dumpFormat'=>true,'dumpOutput'=>true,'editRowPrint'=>true,'editFunctions'=>true,'config'=>true);var$plugins;var$error='';private$hooks=array();function
__construct($tg){if($tg===null){$tg=array();$Ha="adminer-plugins";if(is_dir($Ha)){foreach(glob("$Ha/*.php")as$o)$Ud=include_once"./$o";}$Fd=" href='https://www.adminer.org/plugins/#use'".target_blank();if(file_exists("$Ha.php")){$Ud=include_once"./$Ha.php";if(is_array($Ud)){foreach($Ud
as$sg)$tg[get_class($sg)]=$sg;}else$this->error
.=sprintf('%s must <a%s>return an array</a>.',"<b>$Ha.php</b>",$Fd)."<br>";}foreach(get_declared_classes()as$cb){if(!$tg[$cb]&&preg_match('~^Adminer\w~i',$cb)){$Ug=new
\ReflectionClass($cb);$vb=$Ug->getConstructor();if($vb&&$vb->getNumberOfRequiredParameters())$this->error
.=sprintf('<a%s>Configure</a> %s in %s.',$Fd,"<b>$cb</b>","<b>$Ha.php</b>")."<br>";else$tg[$cb]=new$cb;}}}$this->plugins=$tg;$la=new
Adminer;$tg[]=$la;$Ug=new
\ReflectionObject($la);foreach($Ug->getMethods()as$cf){foreach($tg
as$sg){$C=$cf->getName();if(method_exists($sg,$C))$this->hooks[$C][]=$sg;}}}function
__call($C,array$fg){$ua=array();foreach($fg
as$x=>$X)$ua[]=&$fg[$x];$J=null;foreach($this->hooks[$C]as$sg){$Y=call_user_func_array(array($sg,$C),$ua);if($Y!==null){if(!self::$ta[$C])return$Y;$J=$Y+(array)$J;}}return$J;}}abstract
class
Plugin{protected$translations=array();function
description(){return$this->lang('');}function
screenshot(){return"";}protected
function
lang($u,$uf=null){$ua=func_get_args();$ua[0]=idx($this->translations[LANG],$u)?:$u;return
call_user_func_array('Adminer\lang_format',$ua);}}Adminer::$fe=(function_exists('adminer_object')?adminer_object():(is_dir("adminer-plugins")||file_exists("adminer-plugins.php")?new
Plugins(null):new
Adminer));SqlDriver::$ec=array("server"=>"MySQL / MariaDB")+SqlDriver::$ec;if(!defined('Adminer\DRIVER')){define('Adminer\DRIVER',"server");if(extension_loaded("mysqli")&&$_GET["ext"]!="pdo"){class
Db
extends
\MySQLi{static$fe;var$extension="MySQLi",$flavor='';function
__construct(){parent::init();}function
attach($N,$V,$F){mysqli_report(MYSQLI_REPORT_OFF);list($Jd,$ug)=explode(":",$N,2);$Sh=adminer()->connectSsl();if($Sh)$this->ssl_set($Sh['key'],$Sh['cert'],$Sh['ca'],'','');$J=@$this->real_connect(($N!=""?$Jd:ini_get("mysqli.default_host")),($N.$V!=""?$V:ini_get("mysqli.default_user")),($N.$V.$F!=""?$F:ini_get("mysqli.default_pw")),null,(is_numeric($ug)?intval($ug):ini_get("mysqli.default_port")),(is_numeric($ug)?$ug:null),($Sh?($Sh['verify']!==false?2048:64):0));$this->options(MYSQLI_OPT_LOCAL_INFILE,false);return($J?'':$this->error);}function
set_charset($Ua){if(parent::set_charset($Ua))return
true;parent::set_charset('utf8');return$this->query("SET NAMES $Ua");}function
next_result(){return
self::more_results()&&parent::next_result();}function
quote($Q){return"'".$this->escape_string($Q)."'";}}}elseif(extension_loaded("mysql")&&!((ini_bool("sql.safe_mode")||ini_bool("mysql.allow_local_infile"))&&extension_loaded("pdo_mysql"))){class
Db
extends
SqlDb{private$link;function
attach($N,$V,$F){if(ini_bool("mysql.allow_local_infile"))return
sprintf('Disable %s or enable %s or %s extensions.',"'mysql.allow_local_infile'","MySQLi","PDO_MySQL");$this->link=@mysql_connect(($N!=""?$N:ini_get("mysql.default_host")),("$N$V"!=""?$V:ini_get("mysql.default_user")),("$N$V$F"!=""?$F:ini_get("mysql.default_password")),true,131072);if(!$this->link)return
mysql_error();$this->server_info=mysql_get_server_info($this->link);return'';}function
set_charset($Ua){if(function_exists('mysql_set_charset')){if(mysql_set_charset($Ua,$this->link))return
true;mysql_set_charset('utf8',$this->link);}return$this->query("SET NAMES $Ua");}function
quote($Q){return"'".mysql_real_escape_string($Q,$this->link)."'";}function
select_db($Lb){return
mysql_select_db($Lb,$this->link);}function
query($H,$Ti=false){$I=@($Ti?mysql_unbuffered_query($H,$this->link):mysql_query($H,$this->link));$this->error="";if(!$I){$this->errno=mysql_errno($this->link);$this->error=mysql_error($this->link);return
false;}if($I===true){$this->affected_rows=mysql_affected_rows($this->link);$this->info=mysql_info($this->link);return
true;}return
new
Result($I);}}class
Result{var$num_rows;private$result;private$offset=0;function
__construct($I){$this->result=$I;$this->num_rows=mysql_num_rows($I);}function
fetch_assoc(){return
mysql_fetch_assoc($this->result);}function
fetch_row(){return
mysql_fetch_row($this->result);}function
fetch_field(){$J=mysql_fetch_field($this->result,$this->offset++);$J->orgtable=$J->table;$J->charsetnr=($J->blob?63:0);return$J;}function
__destruct(){mysql_free_result($this->result);}}}elseif(extension_loaded("pdo_mysql")){class
Db
extends
PdoDb{var$extension="PDO_MySQL";function
attach($N,$V,$F){$Lf=array(\PDO::MYSQL_ATTR_LOCAL_INFILE=>false);$Sh=adminer()->connectSsl();if($Sh){if($Sh['key'])$Lf[\PDO::MYSQL_ATTR_SSL_KEY]=$Sh['key'];if($Sh['cert'])$Lf[\PDO::MYSQL_ATTR_SSL_CERT]=$Sh['cert'];if($Sh['ca'])$Lf[\PDO::MYSQL_ATTR_SSL_CA]=$Sh['ca'];if(isset($Sh['verify']))$Lf[\PDO::MYSQL_ATTR_SSL_VERIFY_SERVER_CERT]=$Sh['verify'];}return$this->dsn("mysql:charset=utf8;host=".str_replace(":",";unix_socket=",preg_replace('~:(\d)~',';port=\1',$N)),$V,$F,$Lf);}function
set_charset($Ua){return$this->query("SET NAMES $Ua");}function
select_db($Lb){return$this->query("USE ".idf_escape($Lb));}function
query($H,$Ti=false){$this->pdo->setAttribute(\PDO::MYSQL_ATTR_USE_BUFFERED_QUERY,!$Ti);return
parent::query($H,$Ti);}}}class
Driver
extends
SqlDriver{static$Nc=array("MySQLi","MySQL","PDO_MySQL");static$pe="sql";var$unsigned=array("unsigned","zerofill","unsigned zerofill");var$operators=array("=","<",">","<=",">=","!=","LIKE","LIKE %%","REGEXP","IN","FIND_IN_SET","IS NULL","NOT LIKE","NOT REGEXP","NOT IN","IS NOT NULL","SQL");var$functions=array("char_length","date","from_unixtime","lower","round","floor","ceil","sec_to_time","time_to_sec","upper");var$grouping=array("avg","count","count distinct","group_concat","max","min","sum");static
function
connect($N,$V,$F){$f=parent::connect($N,$V,$F);if(is_string($f)){if(function_exists('iconv')&&!is_utf8($f)&&strlen($kh=iconv("windows-1250","utf-8",$f))>strlen($f))$f=$kh;return$f;}$f->set_charset(charset($f));$f->query("SET sql_quote_show_create = 1, autocommit = 1");$f->flavor=(preg_match('~MariaDB~',$f->server_info)?'maria':'mysql');add_driver(DRIVER,($f->flavor=='maria'?"MariaDB":"MySQL"));return$f;}function
__construct(Db$f){parent::__construct($f);$this->types=array('Numbers'=>array("tinyint"=>3,"smallint"=>5,"mediumint"=>8,"int"=>10,"bigint"=>20,"decimal"=>66,"float"=>12,"double"=>21),'Date and time'=>array("date"=>10,"datetime"=>19,"timestamp"=>19,"time"=>10,"year"=>4),'Strings'=>array("char"=>255,"varchar"=>65535,"tinytext"=>255,"text"=>65535,"mediumtext"=>16777215,"longtext"=>4294967295),'Lists'=>array("enum"=>65535,"set"=>64),'Binary'=>array("bit"=>20,"binary"=>255,"varbinary"=>65535,"tinyblob"=>255,"blob"=>65535,"mediumblob"=>16777215,"longblob"=>4294967295),'Geometry'=>array("geometry"=>0,"point"=>0,"linestring"=>0,"polygon"=>0,"multipoint"=>0,"multilinestring"=>0,"multipolygon"=>0,"geometrycollection"=>0),);$this->insertFunctions=array("char"=>"md5/sha1/password/encrypt/uuid","binary"=>"md5/sha1","date|time"=>"now",);$this->editFunctions=array(number_type()=>"+/-","date"=>"+ interval/- interval","time"=>"addtime/subtime","char|text"=>"concat",);if(min_version('5.7.8',10.2,$f))$this->types['Strings']["json"]=4294967295;if(min_version('',10.7,$f)){$this->types['Strings']["uuid"]=128;$this->insertFunctions['uuid']='uuid';}if(min_version(9,'',$f)){$this->types['Numbers']["vector"]=16383;$this->insertFunctions['vector']='string_to_vector';}if(min_version(5.7,10.2,$f))$this->generated=array("STORED","VIRTUAL");}function
unconvertFunction(array$m){return(preg_match("~binary~",$m["type"])?"<code class='jush-sql'>UNHEX</code>":($m["type"]=="bit"?doc_link(array('sql'=>'bit-value-literals.html'),"<code>b''</code>"):(preg_match("~geometry|point|linestring|polygon~",$m["type"])?"<code class='jush-sql'>GeomFromText</code>":"")));}function
insert($R,array$O){return($O?parent::insert($R,$O):queries("INSERT INTO ".table($R)." ()\nVALUES ()"));}function
insertUpdate($R,array$L,array$G){$e=array_keys(reset($L));$_g="INSERT INTO ".table($R)." (".implode(", ",$e).") VALUES\n";$mj=array();foreach($e
as$x)$mj[$x]="$x = VALUES($x)";$Zh="\nON DUPLICATE KEY UPDATE ".implode(", ",$mj);$mj=array();$y=0;foreach($L
as$O){$Y="(".implode(", ",$O).")";if($mj&&(strlen($_g)+$y+strlen($Y)+strlen($Zh)>1e6)){if(!queries($_g.implode(",\n",$mj).$Zh))return
false;$mj=array();$y=0;}$mj[]=$Y;$y+=strlen($Y)+2;}return
queries($_g.implode(",\n",$mj).$Zh);}function
slowQuery($H,$wi){if(min_version('5.7.8','10.1.2')){if($this->conn->flavor=='maria')return"SET STATEMENT max_statement_time=$wi FOR $H";elseif(preg_match('~^(SELECT\b)(.+)~is',$H,$B))return"$B[1] /*+ MAX_EXECUTION_TIME(".($wi*1000).") */ $B[2]";}}function
convertSearch($u,array$X,array$m){return(preg_match('~char|text|enum|set~',$m["type"])&&!preg_match("~^utf8~",$m["collation"])&&preg_match('~[\x80-\xFF]~',$X['val'])?"CONVERT($u USING ".charset($this->conn).")":$u);}function
warnings(){$I=$this->conn->query("SHOW WARNINGS");if($I&&$I->num_rows){ob_start();print_select_result($I);return
ob_get_clean();}}function
tableHelp($C,$ne=false){$Je=($this->conn->flavor=='maria');if(information_schema(DB))return
strtolower("information-schema-".($Je?"$C-table/":str_replace("_","-",$C)."-table.html"));if(DB=="mysql")return($Je?"mysql$C-table/":"system-schema.html");}function
hasCStyleEscapes(){static$Pa;if($Pa===null){$Qh=get_val("SHOW VARIABLES LIKE 'sql_mode'",1,$this->conn);$Pa=(strpos($Qh,'NO_BACKSLASH_ESCAPES')===false);}return$Pa;}function
engines(){$J=array();foreach(get_rows("SHOW ENGINES")as$K){if(preg_match("~YES|DEFAULT~",$K["Support"]))$J[]=$K["Engine"];}return$J;}}function
idf_escape($u){return"`".str_replace("`","``",$u)."`";}function
table($u){return
idf_escape($u);}function
get_databases($ed){$J=get_session("dbs");if($J===null){$H="SELECT SCHEMA_NAME FROM information_schema.SCHEMATA ORDER BY SCHEMA_NAME";$J=($ed?slow_query($H):get_vals($H));restart_session();set_session("dbs",$J);stop_session();}return$J;}function
limit($H,$Z,$z,$D=0,$xh=" "){return" $H$Z".($z?$xh."LIMIT $z".($D?" OFFSET $D":""):"");}function
limit1($R,$H,$Z,$xh="\n"){return
limit($H,$Z,1,0,$xh);}function
db_collation($j,array$hb){$J=null;$h=get_val("SHOW CREATE DATABASE ".idf_escape($j),1);if(preg_match('~ COLLATE ([^ ]+)~',$h,$B))$J=$B[1];elseif(preg_match('~ CHARACTER SET ([^ ]+)~',$h,$B))$J=$hb[$B[1]][-1];return$J;}function
logged_user(){return
get_val("SELECT USER()");}function
tables_list(){return
get_key_vals("SELECT TABLE_NAME, TABLE_TYPE FROM information_schema.TABLES WHERE TABLE_SCHEMA = DATABASE() ORDER BY TABLE_NAME");}function
count_tables(array$i){$J=array();foreach($i
as$j)$J[$j]=count(get_vals("SHOW TABLES IN ".idf_escape($j)));return$J;}function
table_status($C="",$Rc=false){$J=array();foreach(get_rows($Rc?"SELECT TABLE_NAME AS Name, ENGINE AS Engine, TABLE_COMMENT AS Comment FROM information_schema.TABLES WHERE TABLE_SCHEMA = DATABASE() ".($C!=""?"AND TABLE_NAME = ".q($C):"ORDER BY Name"):"SHOW TABLE STATUS".($C!=""?" LIKE ".q(addcslashes($C,"%_\\")):""))as$K){if($K["Engine"]=="InnoDB")$K["Comment"]=preg_replace('~(?:(.+); )?InnoDB free: .*~','\1',$K["Comment"]);if(!isset($K["Engine"]))$K["Comment"]="";if($C!="")$K["Name"]=$C;$J[$K["Name"]]=$K;}return$J;}function
is_view(array$S){return$S["Engine"]===null;}function
fk_support(array$S){return
preg_match('~InnoDB|IBMDB2I'.(min_version(5.6)?'|NDB':'').'~i',$S["Engine"]);}function
fields($R){$Je=(connection()->flavor=='maria');$J=array();foreach(get_rows("SELECT * FROM information_schema.COLUMNS WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = ".q($R)." ORDER BY ORDINAL_POSITION")as$K){$m=$K["COLUMN_NAME"];$U=$K["COLUMN_TYPE"];$qd=$K["GENERATION_EXPRESSION"];$Oc=$K["EXTRA"];preg_match('~^(VIRTUAL|PERSISTENT|STORED)~',$Oc,$pd);preg_match('~^([^( ]+)(?:\((.+)\))?( unsigned)?( zerofill)?$~',$U,$Me);$k=$K["COLUMN_DEFAULT"];if($k!=""){$me=preg_match('~text|json~',$Me[1]);if(!$Je&&$me)$k=preg_replace("~^(_\w+)?('.*')$~",'\2',stripslashes($k));if($Je||$me){$k=($k=="NULL"?null:preg_replace_callback("~^'(.*)'$~",function($B){return
stripslashes(str_replace("''","'",$B[1]));},$k));}if(!$Je&&preg_match('~binary~',$Me[1])&&preg_match('~^0x(\w*)$~',$k,$B))$k=pack("H*",$B[1]);}$J[$m]=array("field"=>$m,"full_type"=>$U,"type"=>$Me[1],"length"=>$Me[2],"unsigned"=>ltrim($Me[3].$Me[4]),"default"=>($pd?($Je?$qd:stripslashes($qd)):$k),"null"=>($K["IS_NULLABLE"]=="YES"),"auto_increment"=>($Oc=="auto_increment"),"on_update"=>(preg_match('~\bon update (\w+)~i',$Oc,$B)?$B[1]:""),"collation"=>$K["COLLATION_NAME"],"privileges"=>array_flip(explode(",","$K[PRIVILEGES],where,order")),"comment"=>$K["COLUMN_COMMENT"],"primary"=>($K["COLUMN_KEY"]=="PRI"),"generated"=>($pd[1]=="PERSISTENT"?"STORED":$pd[1]),);}return$J;}function
indexes($R,$g=null){$J=array();foreach(get_rows("SHOW INDEX FROM ".table($R),$g)as$K){$C=$K["Key_name"];$J[$C]["type"]=($C=="PRIMARY"?"PRIMARY":($K["Index_type"]=="FULLTEXT"?"FULLTEXT":($K["Non_unique"]?($K["Index_type"]=="SPATIAL"?"SPATIAL":"INDEX"):"UNIQUE")));$J[$C]["columns"][]=$K["Column_name"];$J[$C]["lengths"][]=($K["Index_type"]=="SPATIAL"?null:$K["Sub_part"]);$J[$C]["descs"][]=null;}return$J;}function
foreign_keys($R){static$og='(?:`(?:[^`]|``)+`|"(?:[^"]|"")+")';$J=array();$Bb=get_val("SHOW CREATE TABLE ".table($R),1);if($Bb){preg_match_all("~CONSTRAINT ($og) FOREIGN KEY ?\\(((?:$og,? ?)+)\\) REFERENCES ($og)(?:\\.($og))? \\(((?:$og,? ?)+)\\)(?: ON DELETE (".driver()->onActions."))?(?: ON UPDATE (".driver()->onActions."))?~",$Bb,$Ne,PREG_SET_ORDER);foreach($Ne
as$B){preg_match_all("~$og~",$B[2],$Kh);preg_match_all("~$og~",$B[5],$oi);$J[idf_unescape($B[1])]=array("db"=>idf_unescape($B[4]!=""?$B[3]:$B[4]),"table"=>idf_unescape($B[4]!=""?$B[4]:$B[3]),"source"=>array_map('Adminer\idf_unescape',$Kh[0]),"target"=>array_map('Adminer\idf_unescape',$oi[0]),"on_delete"=>($B[6]?:"RESTRICT"),"on_update"=>($B[7]?:"RESTRICT"),);}}return$J;}function
view($C){return
array("select"=>preg_replace('~^(?:[^`]|`[^`]*`)*\s+AS\s+~isU','',get_val("SHOW CREATE VIEW ".table($C),1)));}function
collations(){$J=array();foreach(get_rows("SHOW COLLATION")as$K){if($K["Default"])$J[$K["Charset"]][-1]=$K["Collation"];else$J[$K["Charset"]][]=$K["Collation"];}ksort($J);foreach($J
as$x=>$X)sort($J[$x]);return$J;}function
information_schema($j){return($j=="information_schema")||(min_version(5.5)&&$j=="performance_schema");}function
error(){return
h(preg_replace('~^You have an error.*syntax to use~U',"Syntax error",connection()->error));}function
create_database($j,$c){return
queries("CREATE DATABASE ".idf_escape($j).($c?" COLLATE ".q($c):""));}function
drop_databases(array$i){$J=apply_queries("DROP DATABASE",$i,'Adminer\idf_escape');restart_session();set_session("dbs",null);return$J;}function
rename_database($C,$c){$J=false;if(create_database($C,$c)){$T=array();$rj=array();foreach(tables_list()as$R=>$U){if($U=='VIEW')$rj[]=$R;else$T[]=$R;}$J=(!$T&&!$rj)||move_tables($T,$rj,$C);drop_databases($J?array(DB):array());}return$J;}function
auto_increment(){$Aa=" PRIMARY KEY";if($_GET["create"]!=""&&$_POST["auto_increment_col"]){foreach(indexes($_GET["create"])as$v){if(in_array($_POST["fields"][$_POST["auto_increment_col"]]["orig"],$v["columns"],true)){$Aa="";break;}if($v["type"]=="PRIMARY")$Aa=" UNIQUE";}}return" AUTO_INCREMENT$Aa";}function
alter_table($R,$C,array$n,array$gd,$mb,$vc,$c,$_a,$kg){$b=array();foreach($n
as$m){if($m[1]){$k=$m[1][3];if(preg_match('~ GENERATED~',$k)){$m[1][3]=(connection()->flavor=='maria'?"":$m[1][2]);$m[1][2]=$k;}$b[]=($R!=""?($m[0]!=""?"CHANGE ".idf_escape($m[0]):"ADD"):" ")." ".implode($m[1]).($R!=""?$m[2]:"");}else$b[]="DROP ".idf_escape($m[0]);}$b=array_merge($b,$gd);$P=($mb!==null?" COMMENT=".q($mb):"").($vc?" ENGINE=".q($vc):"").($c?" COLLATE ".q($c):"").($_a!=""?" AUTO_INCREMENT=$_a":"");if($R=="")return
queries("CREATE TABLE ".table($C)." (\n".implode(",\n",$b)."\n)$P$kg");if($R!=$C)$b[]="RENAME TO ".table($C);if($P)$b[]=ltrim($P);return($b||$kg?queries("ALTER TABLE ".table($R)."\n".implode(",\n",$b).$kg):true);}function
alter_indexes($R,$b){$Ta=array();foreach($b
as$X)$Ta[]=($X[2]=="DROP"?"\nDROP INDEX ".idf_escape($X[1]):"\nADD $X[0] ".($X[0]=="PRIMARY"?"KEY ":"").($X[1]!=""?idf_escape($X[1])." ":"")."(".implode(", ",$X[2]).")");return
queries("ALTER TABLE ".table($R).implode(",",$Ta));}function
truncate_tables(array$T){return
apply_queries("TRUNCATE TABLE",$T);}function
drop_views(array$rj){return
queries("DROP VIEW ".implode(", ",array_map('Adminer\table',$rj)));}function
drop_tables(array$T){return
queries("DROP TABLE ".implode(", ",array_map('Adminer\table',$T)));}function
move_tables(array$T,array$rj,$oi){$Yg=array();foreach($T
as$R)$Yg[]=table($R)." TO ".idf_escape($oi).".".table($R);if(!$Yg||queries("RENAME TABLE ".implode(", ",$Yg))){$Tb=array();foreach($rj
as$R)$Tb[table($R)]=view($R);connection()->select_db($oi);$j=idf_escape(DB);foreach($Tb
as$C=>$qj){if(!queries("CREATE VIEW $C AS ".str_replace(" $j."," ",$qj["select"]))||!queries("DROP VIEW $j.$C"))return
false;}return
true;}return
false;}function
copy_tables(array$T,array$rj,$oi){queries("SET sql_mode = 'NO_AUTO_VALUE_ON_ZERO'");foreach($T
as$R){$C=($oi==DB?table("copy_$R"):idf_escape($oi).".".table($R));if(($_POST["overwrite"]&&!queries("\nDROP TABLE IF EXISTS $C"))||!queries("CREATE TABLE $C LIKE ".table($R))||!queries("INSERT INTO $C SELECT * FROM ".table($R)))return
false;foreach(get_rows("SHOW TRIGGERS LIKE ".q(addcslashes($R,"%_\\")))as$K){$Mi=$K["Trigger"];if(!queries("CREATE TRIGGER ".($oi==DB?idf_escape("copy_$Mi"):idf_escape($oi).".".idf_escape($Mi))." $K[Timing] $K[Event] ON $C FOR EACH ROW\n$K[Statement];"))return
false;}}foreach($rj
as$R){$C=($oi==DB?table("copy_$R"):idf_escape($oi).".".table($R));$qj=view($R);if(($_POST["overwrite"]&&!queries("DROP VIEW IF EXISTS $C"))||!queries("CREATE VIEW $C AS $qj[select]"))return
false;}return
true;}function
trigger($C,$R){if($C=="")return
array();$L=get_rows("SHOW TRIGGERS WHERE `Trigger` = ".q($C));return
reset($L);}function
triggers($R){$J=array();foreach(get_rows("SHOW TRIGGERS LIKE ".q(addcslashes($R,"%_\\")))as$K)$J[$K["Trigger"]]=array($K["Timing"],$K["Event"]);return$J;}function
trigger_options(){return
array("Timing"=>array("BEFORE","AFTER"),"Event"=>array("INSERT","UPDATE","DELETE"),"Type"=>array("FOR EACH ROW"),);}function
routine($C,$U){$ra=array("bool","boolean","integer","double precision","real","dec","numeric","fixed","national char","national varchar");$Lh="(?:\\s|/\\*[\s\S]*?\\*/|(?:#|-- )[^\n]*\n?|--\r?\n)";$xc=driver()->enumLength;$Ri="((".implode("|",array_merge(array_keys(driver()->types()),$ra)).")\\b(?:\\s*\\(((?:[^'\")]|$xc)++)\\))?"."\\s*(zerofill\\s*)?(unsigned(?:\\s+zerofill)?)?)(?:\\s*(?:CHARSET|CHARACTER\\s+SET)\\s*['\"]?([^'\"\\s,]+)['\"]?)?";$og="$Lh*(".($U=="FUNCTION"?"":driver()->inout).")?\\s*(?:`((?:[^`]|``)*)`\\s*|\\b(\\S+)\\s+)$Ri";$h=get_val("SHOW CREATE $U ".idf_escape($C),2);preg_match("~\\(((?:$og\\s*,?)*)\\)\\s*".($U=="FUNCTION"?"RETURNS\\s+$Ri\\s+":"")."(.*)~is",$h,$B);$n=array();preg_match_all("~$og\\s*,?~is",$B[1],$Ne,PREG_SET_ORDER);foreach($Ne
as$eg)$n[]=array("field"=>str_replace("``","`",$eg[2]).$eg[3],"type"=>strtolower($eg[5]),"length"=>preg_replace_callback("~$xc~s",'Adminer\normalize_enum',$eg[6]),"unsigned"=>strtolower(preg_replace('~\s+~',' ',trim("$eg[8] $eg[7]"))),"null"=>true,"full_type"=>$eg[4],"inout"=>strtoupper($eg[1]),"collation"=>strtolower($eg[9]),);return
array("fields"=>$n,"comment"=>get_val("SELECT ROUTINE_COMMENT FROM information_schema.ROUTINES WHERE ROUTINE_SCHEMA = DATABASE() AND ROUTINE_NAME = ".q($C)),)+($U!="FUNCTION"?array("definition"=>$B[11]):array("returns"=>array("type"=>$B[12],"length"=>$B[13],"unsigned"=>$B[15],"collation"=>$B[16]),"definition"=>$B[17],"language"=>"SQL",));}function
routines(){return
get_rows("SELECT ROUTINE_NAME AS SPECIFIC_NAME, ROUTINE_NAME, ROUTINE_TYPE, DTD_IDENTIFIER FROM information_schema.ROUTINES WHERE ROUTINE_SCHEMA = DATABASE()");}function
routine_languages(){return
array();}function
routine_id($C,array$K){return
idf_escape($C);}function
last_id($I){return
get_val("SELECT LAST_INSERT_ID()");}function
explain(Db$f,$H){return$f->query("EXPLAIN ".(min_version(5.1)&&!min_version(5.7)?"PARTITIONS ":"").$H);}function
found_rows(array$S,array$Z){return($Z||$S["Engine"]!="InnoDB"?null:$S["Rows"]);}function
create_sql($R,$_a,$Xh){$J=get_val("SHOW CREATE TABLE ".table($R),1);if(!$_a)$J=preg_replace('~ AUTO_INCREMENT=\d+~','',$J);return$J;}function
truncate_sql($R){return"TRUNCATE ".table($R);}function
use_sql($Lb){return"USE ".idf_escape($Lb);}function
trigger_sql($R){$J="";foreach(get_rows("SHOW TRIGGERS LIKE ".q(addcslashes($R,"%_\\")),null,"-- ")as$K)$J
.="\nCREATE TRIGGER ".idf_escape($K["Trigger"])." $K[Timing] $K[Event] ON ".table($K["Table"])." FOR EACH ROW\n$K[Statement];;\n";return$J;}function
show_variables(){return
get_rows("SHOW VARIABLES");}function
show_status(){return
get_rows("SHOW STATUS");}function
process_list(){return
get_rows("SHOW FULL PROCESSLIST");}function
convert_field(array$m){if(preg_match("~binary~",$m["type"]))return"HEX(".idf_escape($m["field"]).")";if($m["type"]=="bit")return"BIN(".idf_escape($m["field"])." + 0)";if(preg_match("~geometry|point|linestring|polygon~",$m["type"]))return(min_version(8)?"ST_":"")."AsWKT(".idf_escape($m["field"]).")";}function
unconvert_field(array$m,$J){if(preg_match("~binary~",$m["type"]))$J="UNHEX($J)";if($m["type"]=="bit")$J="CONVERT(b$J, UNSIGNED)";if(preg_match("~geometry|point|linestring|polygon~",$m["type"])){$_g=(min_version(8)?"ST_":"");$J=$_g."GeomFromText($J, $_g"."SRID($m[field]))";}return$J;}function
support($Sc){return!preg_match("~scheme|sequence|type|view_trigger|materializedview".(min_version(8)?"":"|descidx".(min_version(5.1)?"":"|event|partitioning")).(min_version('8.0.16','10.2.1')?"":"|check")."~",$Sc);}function
kill_process($X){return
queries("KILL ".number($X));}function
connection_id(){return"SELECT CONNECTION_ID()";}function
max_connections(){return
get_val("SELECT @@max_connections");}function
types(){return
array();}function
type_values($t){return"";}function
schemas(){return
array();}function
get_schema(){return"";}function
set_schema($mh,$g=null){return
true;}}define('Adminer\JUSH',Driver::$pe);define('Adminer\SERVER',$_GET[DRIVER]);define('Adminer\DB',$_GET["db"]);define('Adminer\ME',preg_replace('~\?.*~','',relative_uri()).'?'.(sid()?SID.'&':'').(SERVER!==null?DRIVER."=".urlencode(SERVER).'&':'').($_GET["ext"]?"ext=".urlencode($_GET["ext"]).'&':'').(isset($_GET["username"])?"username=".urlencode($_GET["username"]).'&':'').(DB!=""?'db='.urlencode(DB).'&'.(isset($_GET["ns"])?"ns=".urlencode($_GET["ns"])."&":""):''));function
page_header($yi,$l="",$Ma=array(),$zi=""){page_headers();if(is_ajax()&&$l){page_messages($l);exit;}if(!ob_get_level())ob_start('ob_gzhandler',4096);$_i=$yi.($zi!=""?": $zi":"");$Ai=strip_tags($_i.(SERVER!=""&&SERVER!="localhost"?h(" - ".SERVER):"")." - ".adminer()->name());echo'<!DOCTYPE html>
<html lang="en" dir="ltr">
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<meta name="robots" content="noindex">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>',$Ai,'</title>
<link rel="stylesheet" href="',h(preg_replace("~\\?.*~","",ME)."?file=default.css&version=5.2.1"),'">
';$Fb=adminer()->css();$Bd=false;$_d=false;foreach($Fb
as$o){if(strpos($o,"adminer.css")!==false)$Bd=true;if(strpos($o,"adminer-dark.css")!==false)$_d=true;}$Ib=($Bd?($_d?null:false):($_d?:null));$Ve=" media='(prefers-color-scheme: dark)'";if($Ib!==false)echo"<link rel='stylesheet'".($Ib?"":$Ve)." href='".h(preg_replace("~\\?.*~","",ME)."?file=dark.css&version=5.2.1")."'>\n";echo"<meta name='color-scheme' content='".($Ib===null?"light dark":($Ib?"dark":"light"))."'>\n",script_src(preg_replace("~\\?.*~","",ME)."?file=functions.js&version=5.2.1");if(adminer()->head($Ib))echo"<link rel='icon' href='data:image/gif;base64,R0lGODlhEAAQAJEAAAQCBPz+/PwCBAROZCH5BAEAAAAALAAAAAAQABAAAAI2hI+pGO1rmghihiUdvUBnZ3XBQA7f05mOak1RWXrNq5nQWHMKvuoJ37BhVEEfYxQzHjWQ5qIAADs='>\n","<link rel='apple-touch-icon' href='".h(preg_replace("~\\?.*~","",ME)."?file=logo.png&version=5.2.1")."'>\n";foreach($Fb
as$X)echo"<link rel='stylesheet'".(preg_match('~-dark\.~',$X)&&!$Ib?$Ve:"")." href='".h($X)."'>\n";echo"\n<body class='".'ltr'." nojs'>\n";$o=get_temp_dir()."/adminer.version";if(!$_COOKIE["adminer_version"]&&function_exists('openssl_verify')&&file_exists($o)&&filemtime($o)+86400>time()){$pj=unserialize(file_get_contents($o));$Ig="-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAwqWOVuF5uw7/+Z70djoK
RlHIZFZPO0uYRezq90+7Amk+FDNd7KkL5eDve+vHRJBLAszF/7XKXe11xwliIsFs
DFWQlsABVZB3oisKCBEuI71J4kPH8dKGEWR9jDHFw3cWmoH3PmqImX6FISWbG3B8
h7FIx3jEaw5ckVPVTeo5JRm/1DZzJxjyDenXvBQ/6o9DgZKeNDgxwKzH+sw9/YCO
jHnq1cFpOIISzARlrHMa/43YfeNRAm/tsBXjSxembBPo7aQZLAWHmaj5+K19H10B
nCpz9Y++cipkVEiKRGih4ZEvjoFysEOdRLj6WiD/uUNky4xGeA6LaJqh5XpkFkcQ
fQIDAQAB
-----END PUBLIC KEY-----
";if(openssl_verify($pj["version"],base64_decode($pj["signature"]),$Ig)==1)$_COOKIE["adminer_version"]=$pj["version"];}echo
script("mixin(document.body, {onkeydown: bodyKeydown, onclick: bodyClick".(isset($_COOKIE["adminer_version"])?"":", onload: partial(verifyVersion, '".VERSION."', '".js_escape(ME)."', '".get_token()."')")."});
document.body.classList.replace('nojs', 'js');
const offlineMessage = '".js_escape('You are offline.')."';
const thousandsSeparator = '".js_escape(',')."';"),"<div id='help' class='jush-".JUSH." jsonly hidden'></div>\n",script("mixin(qs('#help'), {onmouseover: () => { helpOpen = 1; }, onmouseout: helpMouseout});"),"<div id='content'>\n","<span id='menuopen' class='jsonly'>".icon("move","","menu","")."</span>".script("qs('#menuopen').onclick = event => { qs('#foot').classList.toggle('foot'); event.stopPropagation(); }");if($Ma!==null){$_=substr(preg_replace('~\b(username|db|ns)=[^&]*&~','',ME),0,-1);echo'<p id="breadcrumb"><a href="'.h($_?:".").'">'.get_driver(DRIVER).'</a> » ';$_=substr(preg_replace('~\b(db|ns)=[^&]*&~','',ME),0,-1);$N=adminer()->serverName(SERVER);$N=($N!=""?$N:'Server');if($Ma===false)echo"$N\n";else{echo"<a href='".h($_)."' accesskey='1' title='Alt+Shift+1'>$N</a> » ";if($_GET["ns"]!=""||(DB!=""&&is_array($Ma)))echo'<a href="'.h($_."&db=".urlencode(DB).(support("scheme")?"&ns=":"")).'">'.h(DB).'</a> » ';if(is_array($Ma)){if($_GET["ns"]!="")echo'<a href="'.h(substr(ME,0,-1)).'">'.h($_GET["ns"]).'</a> » ';foreach($Ma
as$x=>$X){$Vb=(is_array($X)?$X[1]:h($X));if($Vb!="")echo"<a href='".h(ME."$x=").urlencode(is_array($X)?$X[0]:$X)."'>$Vb</a> » ";}}echo"$yi\n";}}echo"<h2>$_i</h2>\n","<div id='ajaxstatus' class='jsonly hidden'></div>\n";restart_session();page_messages($l);$i=&get_session("dbs");if(DB!=""&&$i&&!in_array(DB,$i,true))$i=null;stop_session();define('Adminer\PAGE_HEADER',1);}function
page_headers(){header("Content-Type: text/html; charset=utf-8");header("Cache-Control: no-cache");header("X-Frame-Options: deny");header("X-XSS-Protection: 0");header("X-Content-Type-Options: nosniff");header("Referrer-Policy: origin-when-cross-origin");foreach(adminer()->csp(csp())as$Eb){$Dd=array();foreach($Eb
as$x=>$X)$Dd[]="$x $X";header("Content-Security-Policy: ".implode("; ",$Dd));}adminer()->headers();}function
csp(){return
array(array("script-src"=>"'self' 'unsafe-inline' 'nonce-".get_nonce()."' 'strict-dynamic'","connect-src"=>"'self'","frame-src"=>"https://www.adminer.org","object-src"=>"'none'","base-uri"=>"'none'","form-action"=>"'self'",),);}function
get_nonce(){static$rf;if(!$rf)$rf=base64_encode(rand_string());return$rf;}function
page_messages($l){$cj=preg_replace('~^[^?]*~','',$_SERVER["REQUEST_URI"]);$bf=idx($_SESSION["messages"],$cj);if($bf){echo"<div class='message'>".implode("</div>\n<div class='message'>",$bf)."</div>".script("messagesPrint();");unset($_SESSION["messages"][$cj]);}if($l)echo"<div class='error'>$l</div>\n";if(adminer()->error)echo"<div class='error'>".adminer()->error."</div>\n";}function
page_footer($ef=""){echo"</div>\n\n<div id='foot' class='foot'>\n<div id='menu'>\n";adminer()->navigation($ef);echo"</div>\n";if($ef!="auth")echo'<form action="" method="post">
<p class="logout">
<span>',h($_GET["username"])."\n",'</span>
<input type="submit" name="logout" value="Logout" id="logout">
',input_token(),'</form>
';echo"</div>\n\n",script("setupSubmitHighlight(document);");}function
int32($jf){while($jf>=2147483648)$jf-=4294967296;while($jf<=-2147483649)$jf+=4294967296;return(int)$jf;}function
long2str(array$W,$tj){$kh='';foreach($W
as$X)$kh
.=pack('V',$X);if($tj)return
substr($kh,0,end($W));return$kh;}function
str2long($kh,$tj){$W=array_values(unpack('V*',str_pad($kh,4*ceil(strlen($kh)/4),"\0")));if($tj)$W[]=strlen($kh);return$W;}function
xxtea_mx($_j,$zj,$ai,$qe){return
int32((($_j>>5&0x7FFFFFF)^$zj<<2)+(($zj>>3&0x1FFFFFFF)^$_j<<4))^int32(($ai^$zj)+($qe^$_j));}function
encrypt_string($Vh,$x){if($Vh=="")return"";$x=array_values(unpack("V*",pack("H*",md5($x))));$W=str2long($Vh,true);$jf=count($W)-1;$_j=$W[$jf];$zj=$W[0];$Jg=floor(6+52/($jf+1));$ai=0;while($Jg-->0){$ai=int32($ai+0x9E3779B9);$mc=$ai>>2&3;for($cg=0;$cg<$jf;$cg++){$zj=$W[$cg+1];$if=xxtea_mx($_j,$zj,$ai,$x[$cg&3^$mc]);$_j=int32($W[$cg]+$if);$W[$cg]=$_j;}$zj=$W[0];$if=xxtea_mx($_j,$zj,$ai,$x[$cg&3^$mc]);$_j=int32($W[$jf]+$if);$W[$jf]=$_j;}return
long2str($W,false);}function
decrypt_string($Vh,$x){if($Vh=="")return"";if(!$x)return
false;$x=array_values(unpack("V*",pack("H*",md5($x))));$W=str2long($Vh,false);$jf=count($W)-1;$_j=$W[$jf];$zj=$W[0];$Jg=floor(6+52/($jf+1));$ai=int32($Jg*0x9E3779B9);while($ai){$mc=$ai>>2&3;for($cg=$jf;$cg>0;$cg--){$_j=$W[$cg-1];$if=xxtea_mx($_j,$zj,$ai,$x[$cg&3^$mc]);$zj=int32($W[$cg]-$if);$W[$cg]=$zj;}$_j=$W[$jf];$if=xxtea_mx($_j,$zj,$ai,$x[$cg&3^$mc]);$zj=int32($W[0]-$if);$W[0]=$zj;$ai=int32($ai-0x9E3779B9);}return
long2str($W,true);}$qg=array();if($_COOKIE["adminer_permanent"]){foreach(explode(" ",$_COOKIE["adminer_permanent"])as$X){list($x)=explode(":",$X);$qg[$x]=$X;}}function
add_invalid_login(){$Fa=get_temp_dir()."/adminer.invalid";foreach(glob("$Fa*")?:array($Fa)as$o){$q=file_open_lock($o);if($q)break;}if(!$q)$q=file_open_lock("$Fa-".rand_string());if(!$q)return;$ie=unserialize(stream_get_contents($q));$vi=time();if($ie){foreach($ie
as$je=>$X){if($X[0]<$vi)unset($ie[$je]);}}$he=&$ie[adminer()->bruteForceKey()];if(!$he)$he=array($vi+30*60,0);$he[1]++;file_write_unlock($q,serialize($ie));}function
check_invalid_login(array&$qg){$ie=array();foreach(glob(get_temp_dir()."/adminer.invalid*")as$o){$q=file_open_lock($o);if($q){$ie=unserialize(stream_get_contents($q));file_unlock($q);break;}}$he=idx($ie,adminer()->bruteForceKey(),array());$qf=($he[1]>29?$he[0]-time():0);if($qf>0)auth_error(lang_format(array('Too many unsuccessful logins, try again in %d minute.','Too many unsuccessful logins, try again in %d minutes.'),ceil($qf/60)),$qg);}$za=$_POST["auth"];if($za){session_regenerate_id();$oj=$za["driver"];$N=$za["server"];$V=$za["username"];$F=(string)$za["password"];$j=$za["db"];set_password($oj,$N,$V,$F);$_SESSION["db"][$oj][$N][$V][$j]=true;if($za["permanent"]){$x=implode("-",array_map('base64_encode',array($oj,$N,$V,$j)));$Dg=adminer()->permanentLogin(true);$qg[$x]="$x:".base64_encode($Dg?encrypt_string($F,$Dg):"");cookie("adminer_permanent",implode(" ",$qg));}if(count($_POST)==1||DRIVER!=$oj||SERVER!=$N||$_GET["username"]!==$V||DB!=$j)redirect(auth_url($oj,$N,$V,$j));}elseif($_POST["logout"]&&(!$_SESSION["token"]||verify_token())){foreach(array("pwds","db","dbs","queries")as$x)set_session($x,null);unset_permanent($qg);redirect(substr(preg_replace('~\b(username|db|ns)=[^&]*&~','',ME),0,-1),'Logout successful.'.' '.'Thanks for using Adminer, consider <a href="https://www.adminer.org/en/donation/">donating</a>.');}elseif($qg&&!$_SESSION["pwds"]){session_regenerate_id();$Dg=adminer()->permanentLogin();foreach($qg
as$x=>$X){list(,$bb)=explode(":",$X);list($oj,$N,$V,$j)=array_map('base64_decode',explode("-",$x));set_password($oj,$N,$V,decrypt_string(base64_decode($bb),$Dg));$_SESSION["db"][$oj][$N][$V][$j]=true;}}function
unset_permanent(array&$qg){foreach($qg
as$x=>$X){list($oj,$N,$V,$j)=array_map('base64_decode',explode("-",$x));if($oj==DRIVER&&$N==SERVER&&$V==$_GET["username"]&&$j==DB)unset($qg[$x]);}cookie("adminer_permanent",implode(" ",$qg));}function
auth_error($l,array&$qg){$Ch=session_name();if(isset($_GET["username"])){header("HTTP/1.1 403 Forbidden");if(($_COOKIE[$Ch]||$_GET[$Ch])&&!$_SESSION["token"])$l='Session expired, please login again.';else{restart_session();add_invalid_login();$F=get_password();if($F!==null){if($F===false)$l
.=($l?'<br>':'').sprintf('Master password expired. <a href="https://www.adminer.org/en/extension/"%s>Implement</a> %s method to make it permanent.',target_blank(),'<code>permanentLogin()</code>');set_password(DRIVER,SERVER,$_GET["username"],null);}unset_permanent($qg);}}if(!$_COOKIE[$Ch]&&$_GET[$Ch]&&ini_bool("session.use_only_cookies"))$l='Session support must be enabled.';$fg=session_get_cookie_params();cookie("adminer_key",($_COOKIE["adminer_key"]?:rand_string()),$fg["lifetime"]);if(!$_SESSION["token"])$_SESSION["token"]=rand(1,1e6);page_header('Login',$l,null);echo"<form action='' method='post'>\n","<div>";if(hidden_fields($_POST,array("auth")))echo"<p class='message'>".'The action will be performed after successful login with the same credentials.'."\n";echo"</div>\n";adminer()->loginForm();echo"</form>\n";page_footer("auth");exit;}if(isset($_GET["username"])&&!class_exists('Adminer\Db')){unset($_SESSION["pwds"][DRIVER]);unset_permanent($qg);page_header('No extension',sprintf('None of the supported PHP extensions (%s) are available.',implode(", ",Driver::$Nc)),false);page_footer("auth");exit;}$f='';if(isset($_GET["username"])&&is_string(get_password())){list($Jd,$ug)=explode(":",SERVER,2);if(preg_match('~^\s*([-+]?\d+)~',$ug,$B)&&($B[1]<1024||$B[1]>65535))auth_error('Connecting to privileged ports is not allowed.',$qg);check_invalid_login($qg);$Db=adminer()->credentials();$f=Driver::connect($Db[0],$Db[1],$Db[2]);if(is_object($f)){Db::$fe=$f;Driver::$fe=new
Driver($f);if($f->flavor)save_settings(array("vendor-".DRIVER."-".SERVER=>get_driver(DRIVER)));}}$He=null;if(!is_object($f)||($He=adminer()->login($_GET["username"],get_password()))!==true){$l=(is_string($f)?nl_br(h($f)):(is_string($He)?$He:'Invalid credentials.')).(preg_match('~^ | $~',get_password())?'<br>'.'There is a space in the input password which might be the cause.':'');auth_error($l,$qg);}if($_POST["logout"]&&$_SESSION["token"]&&!verify_token()){page_header('Logout','Invalid CSRF token. Send the form again.');page_footer("db");exit;}if(!$_SESSION["token"])$_SESSION["token"]=rand(1,1e6);stop_session(true);if($za&&$_POST["token"])$_POST["token"]=get_token();$l='';if($_POST){if(!verify_token()){$ae="max_input_vars";$Te=ini_get($ae);if(extension_loaded("suhosin")){foreach(array("suhosin.request.max_vars","suhosin.post.max_vars")as$x){$X=ini_get($x);if($X&&(!$Te||$X<$Te)){$ae=$x;$Te=$X;}}}$l=(!$_POST["token"]&&$Te?sprintf('Maximum number of allowed fields exceeded. Please increase %s.',"'$ae'"):'Invalid CSRF token. Send the form again.'.' '.'If you did not send this request from Adminer then close this page.');}}elseif($_SERVER["REQUEST_METHOD"]=="POST"){$l=sprintf('Too big POST data. Reduce the data or increase the %s configuration directive.',"'post_max_size'");if(isset($_GET["sql"]))$l
.=' '.'You can upload a big SQL file via FTP and import it from server.';}function
print_select_result($I,$g=null,array$Rf=array(),$z=0){$Ge=array();$w=array();$e=array();$Ka=array();$Si=array();$J=array();for($s=0;(!$z||$s<$z)&&($K=$I->fetch_row());$s++){if(!$s){echo"<div class='scrollable'>\n","<table class='nowrap odds'>\n","<thead><tr>";for($oe=0;$oe<count($K);$oe++){$m=$I->fetch_field();$C=$m->name;$Qf=(isset($m->orgtable)?$m->orgtable:"");$Pf=(isset($m->orgname)?$m->orgname:$C);if($Rf&&JUSH=="sql")$Ge[$oe]=($C=="table"?"table=":($C=="possible_keys"?"indexes=":null));elseif($Qf!=""){if(isset($m->table))$J[$m->table]=$Qf;if(!isset($w[$Qf])){$w[$Qf]=array();foreach(indexes($Qf,$g)as$v){if($v["type"]=="PRIMARY"){$w[$Qf]=array_flip($v["columns"]);break;}}$e[$Qf]=$w[$Qf];}if(isset($e[$Qf][$Pf])){unset($e[$Qf][$Pf]);$w[$Qf][$Pf]=$oe;$Ge[$oe]=$Qf;}}if($m->charsetnr==63)$Ka[$oe]=true;$Si[$oe]=$m->type;echo"<th".($Qf!=""||$m->name!=$Pf?" title='".h(($Qf!=""?"$Qf.":"").$Pf)."'":"").">".h($C).($Rf?doc_link(array('sql'=>"explain-output.html#explain_".strtolower($C),'mariadb'=>"explain/#the-columns-in-explain-select",)):"");}echo"</thead>\n";}echo"<tr>";foreach($K
as$x=>$X){$_="";if(isset($Ge[$x])&&!$e[$Ge[$x]]){if($Rf&&JUSH=="sql"){$R=$K[array_search("table=",$Ge)];$_=ME.$Ge[$x].urlencode($Rf[$R]!=""?$Rf[$R]:$R);}else{$_=ME."edit=".urlencode($Ge[$x]);foreach($w[$Ge[$x]]as$fb=>$oe)$_
.="&where".urlencode("[".bracket_escape($fb)."]")."=".urlencode($K[$oe]);}}elseif(is_url($X))$_=$X;if($X===null)$X="<i>NULL</i>";elseif($Ka[$x]&&!is_utf8($X))$X="<i>".lang_format(array('%d byte','%d bytes'),strlen($X))."</i>";else{$X=h($X);if($Si[$x]==254)$X="<code>$X</code>";}if($_)$X="<a href='".h($_)."'".(is_url($_)?target_blank():'').">$X</a>";echo"<td".($Si[$x]<=9||$Si[$x]==246?" class='number'":"").">$X";}}echo($s?"</table>\n</div>":"<p class='message'>".'No rows.')."\n";return$J;}function
referencable_primary($vh){$J=array();foreach(table_status('',true)as$fi=>$R){if($fi!=$vh&&fk_support($R)){foreach(fields($fi)as$m){if($m["primary"]){if($J[$fi]){unset($J[$fi]);break;}$J[$fi]=$m;}}}}return$J;}function
textarea($C,$Y,$L=10,$ib=80){echo"<textarea name='".h($C)."' rows='$L' cols='$ib' class='sqlarea jush-".JUSH."' spellcheck='false' wrap='off'>";if(is_array($Y)){foreach($Y
as$X)echo
h($X[0])."\n\n\n";}else
echo
h($Y);echo"</textarea>";}function
select_input($ya,array$Lf,$Y="",$Ff="",$rg=""){$ni=($Lf?"select":"input");return"<$ni$ya".($Lf?"><option value=''>$rg".optionlist($Lf,$Y,true)."</select>":" size='10' value='".h($Y)."' placeholder='$rg'>").($Ff?script("qsl('$ni').onchange = $Ff;",""):"");}function
json_row($x,$X=null){static$Yc=true;if($Yc)echo"{";if($x!=""){echo($Yc?"":",")."\n\t\"".addcslashes($x,"\r\n\t\"\\/").'": '.($X!==null?'"'.addcslashes($X,"\r\n\"\\/").'"':'null');$Yc=false;}else{echo"\n}\n";$Yc=true;}}function
edit_type($x,array$m,array$hb,array$id=array(),array$Pc=array()){$U=$m["type"];echo"<td><select name='".h($x)."[type]' class='type' aria-labelledby='label-type'>";if($U&&!array_key_exists($U,driver()->types())&&!isset($id[$U])&&!in_array($U,$Pc))$Pc[]=$U;$Wh=driver()->structuredTypes();if($id)$Wh['Foreign keys']=$id;echo
optionlist(array_merge($Pc,$Wh),$U),"</select><td>","<input name='".h($x)."[length]' value='".h($m["length"])."' size='3'".(!$m["length"]&&preg_match('~var(char|binary)$~',$U)?" class='required'":"")." aria-labelledby='label-length'>","<td class='options'>",($hb?"<input list='collations' name='".h($x)."[collation]'".(preg_match('~(char|text|enum|set)$~',$U)?"":" class='hidden'")." value='".h($m["collation"])."' placeholder='(".'collation'.")'>":''),(driver()->unsigned?"<select name='".h($x)."[unsigned]'".(!$U||preg_match(number_type(),$U)?"":" class='hidden'").'><option>'.optionlist(driver()->unsigned,$m["unsigned"]).'</select>':''),(isset($m['on_update'])?"<select name='".h($x)."[on_update]'".(preg_match('~timestamp|datetime~',$U)?"":" class='hidden'").'>'.optionlist(array(""=>"(".'ON UPDATE'.")","CURRENT_TIMESTAMP"),(preg_match('~^CURRENT_TIMESTAMP~i',$m["on_update"])?"CURRENT_TIMESTAMP":$m["on_update"])).'</select>':''),($id?"<select name='".h($x)."[on_delete]'".(preg_match("~`~",$U)?"":" class='hidden'")."><option value=''>(".'ON DELETE'.")".optionlist(explode("|",driver()->onActions),$m["on_delete"])."</select> ":" ");}function
get_partitions_info($R){$md="FROM information_schema.PARTITIONS WHERE TABLE_SCHEMA = ".q(DB)." AND TABLE_NAME = ".q($R);$I=connection()->query("SELECT PARTITION_METHOD, PARTITION_EXPRESSION, PARTITION_ORDINAL_POSITION $md ORDER BY PARTITION_ORDINAL_POSITION DESC LIMIT 1");$J=array();list($J["partition_by"],$J["partition"],$J["partitions"])=$I->fetch_row();$lg=get_key_vals("SELECT PARTITION_NAME, PARTITION_DESCRIPTION $md AND PARTITION_NAME != '' ORDER BY PARTITION_ORDINAL_POSITION");$J["partition_names"]=array_keys($lg);$J["partition_values"]=array_values($lg);return$J;}function
process_length($y){$zc=driver()->enumLength;return(preg_match("~^\\s*\\(?\\s*$zc(?:\\s*,\\s*$zc)*+\\s*\\)?\\s*\$~",$y)&&preg_match_all("~$zc~",$y,$Ne)?"(".implode(",",$Ne[0]).")":preg_replace('~^[0-9].*~','(\0)',preg_replace('~[^-0-9,+()[\]]~','',$y)));}function
process_type(array$m,$gb="COLLATE"){return" $m[type]".process_length($m["length"]).(preg_match(number_type(),$m["type"])&&in_array($m["unsigned"],driver()->unsigned)?" $m[unsigned]":"").(preg_match('~char|text|enum|set~',$m["type"])&&$m["collation"]?" $gb ".(JUSH=="mssql"?$m["collation"]:q($m["collation"])):"");}function
process_field(array$m,array$Qi){if($m["on_update"])$m["on_update"]=str_ireplace("current_timestamp()","CURRENT_TIMESTAMP",$m["on_update"]);return
array(idf_escape(trim($m["field"])),process_type($Qi),($m["null"]?" NULL":" NOT NULL"),default_value($m),(preg_match('~timestamp|datetime~',$m["type"])&&$m["on_update"]?" ON UPDATE $m[on_update]":""),(support("comment")&&$m["comment"]!=""?" COMMENT ".q($m["comment"]):""),($m["auto_increment"]?auto_increment():null),);}function
default_value(array$m){$k=$m["default"];$pd=$m["generated"];return($k===null?"":(in_array($pd,driver()->generated)?(JUSH=="mssql"?" AS ($k)".($pd=="VIRTUAL"?"":" $pd")."":" GENERATED ALWAYS AS ($k) $pd"):" DEFAULT ".(!preg_match('~^GENERATED ~i',$k)&&(preg_match('~char|binary|text|json|enum|set~',$m["type"])||preg_match('~^(?![a-z])~i',$k))?(JUSH=="sql"&&preg_match('~text|json~',$m["type"])?"(".q($k).")":q($k)):str_ireplace("current_timestamp()","CURRENT_TIMESTAMP",(JUSH=="sqlite"?"($k)":$k)))));}function
type_class($U){foreach(array('char'=>'text','date'=>'time|year','binary'=>'blob','enum'=>'set',)as$x=>$X){if(preg_match("~$x|$X~",$U))return" class='$x'";}}function
edit_fields(array$n,array$hb,$U="TABLE",array$id=array()){$n=array_values($n);$Qb=(($_POST?$_POST["defaults"]:get_setting("defaults"))?"":" class='hidden'");$nb=(($_POST?$_POST["comments"]:get_setting("comments"))?"":" class='hidden'");echo"<thead><tr>\n",($U=="PROCEDURE"?"<td>":""),"<th id='label-name'>".($U=="TABLE"?'Column name':'Parameter name'),"<td id='label-type'>".'Type'."<textarea id='enum-edit' rows='4' cols='12' wrap='off' style='display: none;'></textarea>".script("qs('#enum-edit').onblur = editingLengthBlur;"),"<td id='label-length'>".'Length',"<td>".'Options';if($U=="TABLE")echo"<td id='label-null'>NULL\n","<td><input type='radio' name='auto_increment_col' value=''><abbr id='label-ai' title='".'Auto Increment'."'>AI</abbr>",doc_link(array('sql'=>"example-auto-increment.html",'mariadb'=>"auto_increment/",'sqlite'=>"autoinc.html",'pgsql'=>"datatype-numeric.html#DATATYPE-SERIAL",'mssql'=>"t-sql/statements/create-table-transact-sql-identity-property",)),"<td id='label-default'$Qb>".'Default value',(support("comment")?"<td id='label-comment'$nb>".'Comment':"");echo"<td>".icon("plus","add[".(support("move_col")?0:count($n))."]","+",'Add next'),"</thead>\n<tbody>\n",script("mixin(qsl('tbody'), {onclick: editingClick, onkeydown: editingKeydown, oninput: editingInput});");foreach($n
as$s=>$m){$s++;$Sf=$m[($_POST?"orig":"field")];$bc=(isset($_POST["add"][$s-1])||(isset($m["field"])&&!idx($_POST["drop_col"],$s)))&&(support("drop_col")||$Sf=="");echo"<tr".($bc?"":" style='display: none;'").">\n",($U=="PROCEDURE"?"<td>".html_select("fields[$s][inout]",explode("|",driver()->inout),$m["inout"]):"")."<th>";if($bc)echo"<input name='fields[$s][field]' value='".h($m["field"])."' data-maxlength='64' autocapitalize='off' aria-labelledby='label-name'>";echo
input_hidden("fields[$s][orig]",$Sf);edit_type("fields[$s]",$m,$hb,$id);if($U=="TABLE")echo"<td>".checkbox("fields[$s][null]",1,$m["null"],"","","block","label-null"),"<td><label class='block'><input type='radio' name='auto_increment_col' value='$s'".($m["auto_increment"]?" checked":"")." aria-labelledby='label-ai'></label>","<td$Qb>".(driver()->generated?html_select("fields[$s][generated]",array_merge(array("","DEFAULT"),driver()->generated),$m["generated"])." ":checkbox("fields[$s][generated]",1,$m["generated"],"","","","label-default")),"<input name='fields[$s][default]' value='".h($m["default"])."' aria-labelledby='label-default'>",(support("comment")?"<td$nb><input name='fields[$s][comment]' value='".h($m["comment"])."' data-maxlength='".(min_version(5.5)?1024:255)."' aria-labelledby='label-comment'>":"");echo"<td>",(support("move_col")?icon("plus","add[$s]","+",'Add next')." ".icon("up","up[$s]","↑",'Move up')." ".icon("down","down[$s]","↓",'Move down')." ":""),($Sf==""||support("drop_col")?icon("cross","drop_col[$s]","x",'Remove'):"");}}function
process_fields(array&$n){$D=0;if($_POST["up"]){$ye=0;foreach($n
as$x=>$m){if(key($_POST["up"])==$x){unset($n[$x]);array_splice($n,$ye,0,array($m));break;}if(isset($m["field"]))$ye=$D;$D++;}}elseif($_POST["down"]){$kd=false;foreach($n
as$x=>$m){if(isset($m["field"])&&$kd){unset($n[key($_POST["down"])]);array_splice($n,$D,0,array($kd));break;}if(key($_POST["down"])==$x)$kd=$m;$D++;}}elseif($_POST["add"]){$n=array_values($n);array_splice($n,key($_POST["add"]),0,array(array()));}elseif(!$_POST["drop_col"])return
false;return
true;}function
normalize_enum(array$B){$X=$B[0];return"'".str_replace("'","''",addcslashes(stripcslashes(str_replace($X[0].$X[0],$X[0],substr($X,1,-1))),'\\'))."'";}function
grant($rd,array$Fg,$e,$Cf){if(!$Fg)return
true;if($Fg==array("ALL PRIVILEGES","GRANT OPTION"))return($rd=="GRANT"?queries("$rd ALL PRIVILEGES$Cf WITH GRANT OPTION"):queries("$rd ALL PRIVILEGES$Cf")&&queries("$rd GRANT OPTION$Cf"));return
queries("$rd ".preg_replace('~(GRANT OPTION)\([^)]*\)~','\1',implode("$e, ",$Fg).$e).$Cf);}function
drop_create($fc,$h,$hc,$ri,$jc,$A,$af,$Ye,$Ze,$_f,$nf){if($_POST["drop"])query_redirect($fc,$A,$af);elseif($_f=="")query_redirect($h,$A,$Ze);elseif($_f!=$nf){$Cb=queries($h);queries_redirect($A,$Ye,$Cb&&queries($fc));if($Cb)queries($hc);}else
queries_redirect($A,$Ye,queries($ri)&&queries($jc)&&queries($fc)&&queries($h));}function
create_trigger($Cf,array$K){$xi=" $K[Timing] $K[Event]".(preg_match('~ OF~',$K["Event"])?" $K[Of]":"");return"CREATE TRIGGER ".idf_escape($K["Trigger"]).(JUSH=="mssql"?$Cf.$xi:$xi.$Cf).rtrim(" $K[Type]\n$K[Statement]",";").";";}function
create_routine($gh,array$K){$O=array();$n=(array)$K["fields"];ksort($n);foreach($n
as$m){if($m["field"]!="")$O[]=(preg_match("~^(".driver()->inout.")\$~",$m["inout"])?"$m[inout] ":"").idf_escape($m["field"]).process_type($m,"CHARACTER SET");}$Sb=rtrim($K["definition"],";");return"CREATE $gh ".idf_escape(trim($K["name"]))." (".implode(", ",$O).")".($gh=="FUNCTION"?" RETURNS".process_type($K["returns"],"CHARACTER SET"):"").($K["language"]?" LANGUAGE $K[language]":"").(JUSH=="pgsql"?" AS ".q($Sb):"\n$Sb;");}function
remove_definer($H){return
preg_replace('~^([A-Z =]+) DEFINER=`'.preg_replace('~@(.*)~','`@`(%|\1)',logged_user()).'`~','\1',$H);}function
format_foreign_key(array$p){$j=$p["db"];$sf=$p["ns"];return" FOREIGN KEY (".implode(", ",array_map('Adminer\idf_escape',$p["source"])).") REFERENCES ".($j!=""&&$j!=$_GET["db"]?idf_escape($j).".":"").($sf!=""&&$sf!=$_GET["ns"]?idf_escape($sf).".":"").idf_escape($p["table"])." (".implode(", ",array_map('Adminer\idf_escape',$p["target"])).")".(preg_match("~^(".driver()->onActions.")\$~",$p["on_delete"])?" ON DELETE $p[on_delete]":"").(preg_match("~^(".driver()->onActions.")\$~",$p["on_update"])?" ON UPDATE $p[on_update]":"");}function
tar_file($o,$Bi){$J=pack("a100a8a8a8a12a12",$o,644,0,0,decoct($Bi->size),decoct(time()));$ab=8*32;for($s=0;$s<strlen($J);$s++)$ab+=ord($J[$s]);$J
.=sprintf("%06o",$ab)."\0 ";echo$J,str_repeat("\0",512-strlen($J));$Bi->send();echo
str_repeat("\0",511-($Bi->size+511)%512);}function
ini_bytes($ae){$X=ini_get($ae);switch(strtolower(substr($X,-1))){case'g':$X=(int)$X*1024;case'm':$X=(int)$X*1024;case'k':$X=(int)$X*1024;}return$X;}function
doc_link(array$ng,$si="<sup>?</sup>"){$Ah=connection()->server_info;$pj=preg_replace('~^(\d\.?\d).*~s','\1',$Ah);$ej=array('sql'=>"https://dev.mysql.com/doc/refman/$pj/en/",'sqlite'=>"https://www.sqlite.org/",'pgsql'=>"https://www.postgresql.org/docs/".(connection()->flavor=='cockroach'?"current":$pj)."/",'mssql'=>"https://learn.microsoft.com/en-us/sql/",'oracle'=>"https://www.oracle.com/pls/topic/lookup?ctx=db".preg_replace('~^.* (\d+)\.(\d+)\.\d+\.\d+\.\d+.*~s','\1\2',$Ah)."&id=",);if(connection()->flavor=='maria'){$ej['sql']="https://mariadb.com/kb/en/";$ng['sql']=(isset($ng['mariadb'])?$ng['mariadb']:str_replace(".html","/",$ng['sql']));}return($ng[JUSH]?"<a href='".h($ej[JUSH].$ng[JUSH].(JUSH=='mssql'?"?view=sql-server-ver$pj":""))."'".target_blank().">$si</a>":"");}function
db_size($j){if(!connection()->select_db($j))return"?";$J=0;foreach(table_status()as$S)$J+=$S["Data_length"]+$S["Index_length"];return
format_number($J);}function
set_utf8mb4($h){static$O=false;if(!$O&&preg_match('~\butf8mb4~i',$h)){$O=true;echo"SET NAMES ".charset(connection()).";\n\n";}}if(isset($_GET["status"]))$_GET["variables"]=$_GET["status"];if(isset($_GET["import"]))$_GET["sql"]=$_GET["import"];if(!(DB!=""?connection()->select_db(DB):isset($_GET["sql"])||isset($_GET["dump"])||isset($_GET["database"])||isset($_GET["processlist"])||isset($_GET["privileges"])||isset($_GET["user"])||isset($_GET["variables"])||$_GET["script"]=="connect"||$_GET["script"]=="kill")){if(DB!=""||$_GET["refresh"]){restart_session();set_session("dbs",null);}if(DB!=""){header("HTTP/1.1 404 Not Found");page_header('Database'.": ".h(DB),'Invalid database.',true);}else{if($_POST["db"]&&!$l)queries_redirect(substr(ME,0,-1),'Databases have been dropped.',drop_databases($_POST["db"]));page_header('Select database',$l,false);echo"<p class='links'>\n";foreach(array('database'=>'Create database','privileges'=>'Privileges','processlist'=>'Process list','variables'=>'Variables','status'=>'Status',)as$x=>$X){if(support($x))echo"<a href='".h(ME)."$x='>$X</a>\n";}echo"<p>".sprintf('%s version: %s through PHP extension %s',get_driver(DRIVER),"<b>".h(connection()->server_info)."</b>","<b>".connection()->extension."</b>")."\n","<p>".sprintf('Logged as: %s',"<b>".h(logged_user())."</b>")."\n";$i=adminer()->databases();if($i){$oh=support("scheme");$hb=collations();echo"<form action='' method='post'>\n","<table class='checkable odds'>\n",script("mixin(qsl('table'), {onclick: tableClick, ondblclick: partialArg(tableClick, true)});"),"<thead><tr>".(support("database")?"<td>":"")."<th>".'Database'.(get_session("dbs")!==null?" - <a href='".h(ME)."refresh=1'>".'Refresh'."</a>":"")."<td>".'Collation'."<td>".'Tables'."<td>".'Size'." - <a href='".h(ME)."dbsize=1'>".'Compute'."</a>".script("qsl('a').onclick = partial(ajaxSetHtml, '".js_escape(ME)."script=connect');","")."</thead>\n";$i=($_GET["dbsize"]?count_tables($i):array_flip($i));foreach($i
as$j=>$T){$fh=h(ME)."db=".urlencode($j);$t=h("Db-".$j);echo"<tr>".(support("database")?"<td>".checkbox("db[]",$j,in_array($j,(array)$_POST["db"]),"","","",$t):""),"<th><a href='$fh' id='$t'>".h($j)."</a>";$c=h(db_collation($j,$hb));echo"<td>".(support("database")?"<a href='$fh".($oh?"&amp;ns=":"")."&amp;database=' title='".'Alter database'."'>$c</a>":$c),"<td align='right'><a href='$fh&amp;schema=' id='tables-".h($j)."' title='".'Database schema'."'>".($_GET["dbsize"]?$T:"?")."</a>","<td align='right' id='size-".h($j)."'>".($_GET["dbsize"]?db_size($j):"?"),"\n";}echo"</table>\n",(support("database")?"<div class='footer'><div>\n"."<fieldset><legend>".'Selected'." <span id='selected'></span></legend><div>\n".input_hidden("all").script("qsl('input').onclick = function () { selectCount('selected', formChecked(this, /^db/)); };")."<input type='submit' name='drop' value='".'Drop'."'>".confirm()."\n"."</div></fieldset>\n"."</div></div>\n":""),input_token(),"</form>\n",script("tableCheck();");}if(!empty(adminer()->plugins)){echo"<div class='plugins'>\n","<h3>".'Loaded plugins'."</h3>\n<ul>\n";foreach(adminer()->plugins
as$sg){$Wb=(method_exists($sg,'description')?$sg->description():"");if(!$Wb){$Ug=new
\ReflectionObject($sg);if(preg_match('~^/[\s*]+(.+)~',$Ug->getDocComment(),$B))$Wb=$B[1];}$ph=(method_exists($sg,'screenshot')?$sg->screenshot():"");echo"<li><b>".get_class($sg)."</b>".h($Wb?": $Wb":"").($ph?" (<a href='".h($ph)."'".target_blank().">".'screenshot'."</a>)":"")."\n";}echo"</ul>\n";adminer()->pluginsLinks();echo"</div>\n";}}page_footer("db");exit;}if(support("scheme")){if(DB!=""&&$_GET["ns"]!==""){if(!isset($_GET["ns"]))redirect(preg_replace('~ns=[^&]*&~','',ME)."ns=".get_schema());if(!set_schema($_GET["ns"])){header("HTTP/1.1 404 Not Found");page_header('Schema'.": ".h($_GET["ns"]),'Invalid schema.',true);page_footer("ns");exit;}}}class
TmpFile{private$handler;var$size;function
__construct(){$this->handler=tmpfile();}function
write($xb){$this->size+=strlen($xb);fwrite($this->handler,$xb);}function
send(){fseek($this->handler,0);fpassthru($this->handler);fclose($this->handler);}}if(isset($_GET["select"])&&($_POST["edit"]||$_POST["clone"])&&!$_POST["save"])$_GET["edit"]=$_GET["select"];if(isset($_GET["callf"]))$_GET["call"]=$_GET["callf"];if(isset($_GET["function"]))$_GET["procedure"]=$_GET["function"];if(isset($_GET["download"])){$a=$_GET["download"];$n=fields($a);header("Content-Type: application/octet-stream");header("Content-Disposition: attachment; filename=".friendly_url("$a-".implode("_",$_GET["where"])).".".friendly_url($_GET["field"]));$M=array(idf_escape($_GET["field"]));$I=driver()->select($a,$M,array(where($_GET,$n)),$M);$K=($I?$I->fetch_row():array());echo
driver()->value($K[0],$n[$_GET["field"]]);exit;}elseif(isset($_GET["table"])){$a=$_GET["table"];$n=fields($a);if(!$n)$l=error()?:'No tables.';$S=table_status1($a);$C=adminer()->tableName($S);page_header(($n&&is_view($S)?$S['Engine']=='materialized view'?'Materialized view':'View':'Table').": ".($C!=""?$C:h($a)),$l);$eh=array();foreach($n
as$x=>$m)$eh+=$m["privileges"];adminer()->selectLinks($S,(isset($eh["insert"])||!support("table")?"":null));$mb=$S["Comment"];if($mb!="")echo"<p class='nowrap'>".'Comment'.": ".h($mb)."\n";if($n)adminer()->tableStructurePrint($n,$S);if(support("indexes")&&driver()->supportsIndex($S)){echo"<h3 id='indexes'>".'Indexes'."</h3>\n";$w=indexes($a);if($w)adminer()->tableIndexesPrint($w);echo'<p class="links"><a href="'.h(ME).'indexes='.urlencode($a).'">'.'Alter indexes'."</a>\n";}if(!is_view($S)){if(fk_support($S)){echo"<h3 id='foreign-keys'>".'Foreign keys'."</h3>\n";$id=foreign_keys($a);if($id){echo"<table>\n","<thead><tr><th>".'Source'."<td>".'Target'."<td>".'ON DELETE'."<td>".'ON UPDATE'."<td></thead>\n";foreach($id
as$C=>$p){echo"<tr title='".h($C)."'>","<th><i>".implode("</i>, <i>",array_map('Adminer\h',$p["source"]))."</i>";$_=($p["db"]!=""?preg_replace('~db=[^&]*~',"db=".urlencode($p["db"]),ME):($p["ns"]!=""?preg_replace('~ns=[^&]*~',"ns=".urlencode($p["ns"]),ME):ME));echo"<td><a href='".h($_."table=".urlencode($p["table"]))."'>".($p["db"]!=""&&$p["db"]!=DB?"<b>".h($p["db"])."</b>.":"").($p["ns"]!=""&&$p["ns"]!=$_GET["ns"]?"<b>".h($p["ns"])."</b>.":"").h($p["table"])."</a>","(<i>".implode("</i>, <i>",array_map('Adminer\h',$p["target"]))."</i>)","<td>".h($p["on_delete"]),"<td>".h($p["on_update"]),'<td><a href="'.h(ME.'foreign='.urlencode($a).'&name='.urlencode($C)).'">'.'Alter'.'</a>',"\n";}echo"</table>\n";}echo'<p class="links"><a href="'.h(ME).'foreign='.urlencode($a).'">'.'Add foreign key'."</a>\n";}if(support("check")){echo"<h3 id='checks'>".'Checks'."</h3>\n";$Wa=driver()->checkConstraints($a);if($Wa){echo"<table>\n";foreach($Wa
as$x=>$X)echo"<tr title='".h($x)."'>","<td><code class='jush-".JUSH."'>".h($X),"<td><a href='".h(ME.'check='.urlencode($a).'&name='.urlencode($x))."'>".'Alter'."</a>","\n";echo"</table>\n";}echo'<p class="links"><a href="'.h(ME).'check='.urlencode($a).'">'.'Create check'."</a>\n";}}if(support(is_view($S)?"view_trigger":"trigger")){echo"<h3 id='triggers'>".'Triggers'."</h3>\n";$Pi=triggers($a);if($Pi){echo"<table>\n";foreach($Pi
as$x=>$X)echo"<tr valign='top'><td>".h($X[0])."<td>".h($X[1])."<th>".h($x)."<td><a href='".h(ME.'trigger='.urlencode($a).'&name='.urlencode($x))."'>".'Alter'."</a>\n";echo"</table>\n";}echo'<p class="links"><a href="'.h(ME).'trigger='.urlencode($a).'">'.'Add trigger'."</a>\n";}}elseif(isset($_GET["schema"])){page_header('Database schema',"",array(),h(DB.($_GET["ns"]?".$_GET[ns]":"")));$hi=array();$ii=array();$ca=($_GET["schema"]?:$_COOKIE["adminer_schema-".str_replace(".","_",DB)]);preg_match_all('~([^:]+):([-0-9.]+)x([-0-9.]+)(_|$)~',$ca,$Ne,PREG_SET_ORDER);foreach($Ne
as$s=>$B){$hi[$B[1]]=array($B[2],$B[3]);$ii[]="\n\t'".js_escape($B[1])."': [ $B[2], $B[3] ]";}$Ei=0;$Ga=-1;$mh=array();$Tg=array();$Be=array();$sa=driver()->allFields();foreach(table_status('',true)as$R=>$S){if(is_view($S))continue;$vg=0;$mh[$R]["fields"]=array();foreach($sa[$R]as$m){$vg+=1.25;$m["pos"]=$vg;$mh[$R]["fields"][$m["field"]]=$m;}$mh[$R]["pos"]=($hi[$R]?:array($Ei,0));foreach(adminer()->foreignKeys($R)as$X){if(!$X["db"]){$_e=$Ga;if(idx($hi[$R],1)||idx($hi[$X["table"]],1))$_e=min(idx($hi[$R],1,0),idx($hi[$X["table"]],1,0))-1;else$Ga-=.1;while($Be[(string)$_e])$_e-=.0001;$mh[$R]["references"][$X["table"]][(string)$_e]=array($X["source"],$X["target"]);$Tg[$X["table"]][$R][(string)$_e]=$X["target"];$Be[(string)$_e]=true;}}$Ei=max($Ei,$mh[$R]["pos"][0]+2.5+$vg);}echo'<div id="schema" style="height: ',$Ei,'em;">
<script',nonce(),'>
qs(\'#schema\').onselectstart = () => false;
const tablePos = {',implode(",",$ii)."\n",'};
const em = qs(\'#schema\').offsetHeight / ',$Ei,';
document.onmousemove = schemaMousemove;
document.onmouseup = partialArg(schemaMouseup, \'',js_escape(DB),'\');
</script>
';foreach($mh
as$C=>$R){echo"<div class='table' style='top: ".$R["pos"][0]."em; left: ".$R["pos"][1]."em;'>",'<a href="'.h(ME).'table='.urlencode($C).'"><b>'.h($C)."</b></a>",script("qsl('div').onmousedown = schemaMousedown;");foreach($R["fields"]as$m){$X='<span'.type_class($m["type"]).' title="'.h($m["type"].($m["length"]?"($m[length])":"").($m["null"]?" NULL":'')).'">'.h($m["field"]).'</span>';echo"<br>".($m["primary"]?"<i>$X</i>":$X);}foreach((array)$R["references"]as$pi=>$Vg){foreach($Vg
as$_e=>$Qg){$Ae=$_e-idx($hi[$C],1);$s=0;foreach($Qg[0]as$Kh)echo"\n<div class='references' title='".h($pi)."' id='refs$_e-".($s++)."' style='left: $Ae"."em; top: ".$R["fields"][$Kh]["pos"]."em; padding-top: .5em;'>"."<div style='border-top: 1px solid gray; width: ".(-$Ae)."em;'></div></div>";}}foreach((array)$Tg[$C]as$pi=>$Vg){foreach($Vg
as$_e=>$e){$Ae=$_e-idx($hi[$C],1);$s=0;foreach($e
as$oi)echo"\n<div class='references arrow' title='".h($pi)."' id='refd$_e-".($s++)."' style='left: $Ae"."em; top: ".$R["fields"][$oi]["pos"]."em;'>"."<div style='height: .5em; border-bottom: 1px solid gray; width: ".(-$Ae)."em;'></div>"."</div>";}}echo"\n</div>\n";}foreach($mh
as$C=>$R){foreach((array)$R["references"]as$pi=>$Vg){foreach($Vg
as$_e=>$Qg){$df=$Ei;$Re=-10;foreach($Qg[0]as$x=>$Kh){$wg=$R["pos"][0]+$R["fields"][$Kh]["pos"];$xg=$mh[$pi]["pos"][0]+$mh[$pi]["fields"][$Qg[1][$x]]["pos"];$df=min($df,$wg,$xg);$Re=max($Re,$wg,$xg);}echo"<div class='references' id='refl$_e' style='left: $_e"."em; top: $df"."em; padding: .5em 0;'><div style='border-right: 1px solid gray; margin-top: 1px; height: ".($Re-$df)."em;'></div></div>\n";}}}echo'</div>
<p class="links"><a href="',h(ME."schema=".urlencode($ca)),'" id="schema-link">Permanent link</a>
';}elseif(isset($_GET["dump"])){$a=$_GET["dump"];if($_POST&&!$l){save_settings(array_intersect_key($_POST,array_flip(array("output","format","db_style","types","routines","events","table_style","auto_increment","triggers","data_style"))),"adminer_export");$T=array_flip((array)$_POST["tables"])+array_flip((array)$_POST["data"]);$Lc=dump_headers((count($T)==1?key($T):DB),(DB==""||count($T)>1));$le=preg_match('~sql~',$_POST["format"]);if($le){echo"-- Adminer ".VERSION." ".get_driver(DRIVER)." ".str_replace("\n"," ",connection()->server_info)." dump\n\n";if(JUSH=="sql"){echo"SET NAMES utf8;
SET time_zone = '+00:00';
SET foreign_key_checks = 0;
".($_POST["data_style"]?"SET sql_mode = 'NO_AUTO_VALUE_ON_ZERO';
":"")."
";connection()->query("SET time_zone = '+00:00'");connection()->query("SET sql_mode = ''");}}$Xh=$_POST["db_style"];$i=array(DB);if(DB==""){$i=$_POST["databases"];if(is_string($i))$i=explode("\n",rtrim(str_replace("\r","",$i),"\n"));}foreach((array)$i
as$j){adminer()->dumpDatabase($j);if(connection()->select_db($j)){if($le&&preg_match('~CREATE~',$Xh)&&($h=get_val("SHOW CREATE DATABASE ".idf_escape($j),1))){set_utf8mb4($h);if($Xh=="DROP+CREATE")echo"DROP DATABASE IF EXISTS ".idf_escape($j).";\n";echo"$h;\n";}if($le){if($Xh)echo
use_sql($j).";\n\n";$Zf="";if($_POST["types"]){foreach(types()as$t=>$U){$_c=type_values($t);if($_c)$Zf
.=($Xh!='DROP+CREATE'?"DROP TYPE IF EXISTS ".idf_escape($U).";;\n":"")."CREATE TYPE ".idf_escape($U)." AS ENUM ($_c);\n\n";else$Zf
.="-- Could not export type $U\n\n";}}if($_POST["routines"]){foreach(routines()as$K){$C=$K["ROUTINE_NAME"];$gh=$K["ROUTINE_TYPE"];$h=create_routine($gh,array("name"=>$C)+routine($K["SPECIFIC_NAME"],$gh));set_utf8mb4($h);$Zf
.=($Xh!='DROP+CREATE'?"DROP $gh IF EXISTS ".idf_escape($C).";;\n":"")."$h;\n\n";}}if($_POST["events"]){foreach(get_rows("SHOW EVENTS",null,"-- ")as$K){$h=remove_definer(get_val("SHOW CREATE EVENT ".idf_escape($K["Name"]),3));set_utf8mb4($h);$Zf
.=($Xh!='DROP+CREATE'?"DROP EVENT IF EXISTS ".idf_escape($K["Name"]).";;\n":"")."$h;;\n\n";}}echo($Zf&&JUSH=='sql'?"DELIMITER ;;\n\n$Zf"."DELIMITER ;\n\n":$Zf);}if($_POST["table_style"]||$_POST["data_style"]){$rj=array();foreach(table_status('',true)as$C=>$S){$R=(DB==""||in_array($C,(array)$_POST["tables"]));$Jb=(DB==""||in_array($C,(array)$_POST["data"]));if($R||$Jb){$Bi=null;if($Lc=="tar"){$Bi=new
TmpFile;ob_start(array($Bi,'write'),1e5);}adminer()->dumpTable($C,($R?$_POST["table_style"]:""),(is_view($S)?2:0));if(is_view($S))$rj[]=$C;elseif($Jb){$n=fields($C);adminer()->dumpData($C,$_POST["data_style"],"SELECT *".convert_fields($n,$n)." FROM ".table($C));}if($le&&$_POST["triggers"]&&$R&&($Pi=trigger_sql($C)))echo"\nDELIMITER ;;\n$Pi\nDELIMITER ;\n";if($Lc=="tar"){ob_end_flush();tar_file((DB!=""?"":"$j/")."$C.csv",$Bi);}elseif($le)echo"\n";}}if(function_exists('Adminer\foreign_keys_sql')){foreach(table_status('',true)as$C=>$S){$R=(DB==""||in_array($C,(array)$_POST["tables"]));if($R&&!is_view($S))echo
foreign_keys_sql($C);}}foreach($rj
as$qj)adminer()->dumpTable($qj,$_POST["table_style"],1);if($Lc=="tar")echo
pack("x512");}}}adminer()->dumpFooter();exit;}page_header('Export',$l,($_GET["export"]!=""?array("table"=>$_GET["export"]):array()),h(DB));echo'
<form action="" method="post">
<table class="layout">
';$Nb=array('','USE','DROP+CREATE','CREATE');$ji=array('','DROP+CREATE','CREATE');$Kb=array('','TRUNCATE+INSERT','INSERT');if(JUSH=="sql")$Kb[]='INSERT+UPDATE';$K=get_settings("adminer_export");if(!$K)$K=array("output"=>"text","format"=>"sql","db_style"=>(DB!=""?"":"CREATE"),"table_style"=>"DROP+CREATE","data_style"=>"INSERT");if(!isset($K["events"])){$K["routines"]=$K["events"]=($_GET["dump"]=="");$K["triggers"]=$K["table_style"];}echo"<tr><th>".'Output'."<td>".html_radios("output",adminer()->dumpOutput(),$K["output"])."\n","<tr><th>".'Format'."<td>".html_radios("format",adminer()->dumpFormat(),$K["format"])."\n",(JUSH=="sqlite"?"":"<tr><th>".'Database'."<td>".html_select('db_style',$Nb,$K["db_style"]).(support("type")?checkbox("types",1,$K["types"],'User types'):"").(support("routine")?checkbox("routines",1,$K["routines"],'Routines'):"").(support("event")?checkbox("events",1,$K["events"],'Events'):"")),"<tr><th>".'Tables'."<td>".html_select('table_style',$ji,$K["table_style"]).checkbox("auto_increment",1,$K["auto_increment"],'Auto Increment').(support("trigger")?checkbox("triggers",1,$K["triggers"],'Triggers'):""),"<tr><th>".'Data'."<td>".html_select('data_style',$Kb,$K["data_style"]),'</table>
<p><input type="submit" value="Export">
',input_token(),'
<table>
',script("qsl('table').onclick = dumpClick;");$Ag=array();if(DB!=""){$Ya=($a!=""?"":" checked");echo"<thead><tr>","<th style='text-align: left;'><label class='block'><input type='checkbox' id='check-tables'$Ya>".'Tables'."</label>".script("qs('#check-tables').onclick = partial(formCheck, /^tables\\[/);",""),"<th style='text-align: right;'><label class='block'>".'Data'."<input type='checkbox' id='check-data'$Ya></label>".script("qs('#check-data').onclick = partial(formCheck, /^data\\[/);",""),"</thead>\n";$rj="";$li=tables_list();foreach($li
as$C=>$U){$_g=preg_replace('~_.*~','',$C);$Ya=($a==""||$a==(substr($a,-1)=="%"?"$_g%":$C));$Cg="<tr><td>".checkbox("tables[]",$C,$Ya,$C,"","block");if($U!==null&&!preg_match('~table~i',$U))$rj
.="$Cg\n";else
echo"$Cg<td align='right'><label class='block'><span id='Rows-".h($C)."'></span>".checkbox("data[]",$C,$Ya)."</label>\n";$Ag[$_g]++;}echo$rj;if($li)echo
script("ajaxSetHtml('".js_escape(ME)."script=db');");}else{echo"<thead><tr><th style='text-align: left;'>","<label class='block'><input type='checkbox' id='check-databases'".($a==""?" checked":"").">".'Database'."</label>",script("qs('#check-databases').onclick = partial(formCheck, /^databases\\[/);",""),"</thead>\n";$i=adminer()->databases();if($i){foreach($i
as$j){if(!information_schema($j)){$_g=preg_replace('~_.*~','',$j);echo"<tr><td>".checkbox("databases[]",$j,$a==""||$a=="$_g%",$j,"","block")."\n";$Ag[$_g]++;}}}else
echo"<tr><td><textarea name='databases' rows='10' cols='20'></textarea>";}echo'</table>
</form>
';$Yc=true;foreach($Ag
as$x=>$X){if($x!=""&&$X>1){echo($Yc?"<p>":" ")."<a href='".h(ME)."dump=".urlencode("$x%")."'>".h($x)."</a>";$Yc=false;}}}elseif(isset($_GET["privileges"])){page_header('Privileges');echo'<p class="links"><a href="'.h(ME).'user=">'.'Create user'."</a>";$I=connection()->query("SELECT User, Host FROM mysql.".(DB==""?"user":"db WHERE ".q(DB)." LIKE Db")." ORDER BY Host, User");$rd=$I;if(!$I)$I=connection()->query("SELECT SUBSTRING_INDEX(CURRENT_USER, '@', 1) AS User, SUBSTRING_INDEX(CURRENT_USER, '@', -1) AS Host");echo"<form action=''><p>\n";hidden_fields_get();echo
input_hidden("db",DB),($rd?"":input_hidden("grant")),"<table class='odds'>\n","<thead><tr><th>".'Username'."<th>".'Server'."<th></thead>\n";while($K=$I->fetch_assoc())echo'<tr><td>'.h($K["User"])."<td>".h($K["Host"]).'<td><a href="'.h(ME.'user='.urlencode($K["User"]).'&host='.urlencode($K["Host"])).'">'.'Edit'."</a>\n";if(!$rd||DB!="")echo"<tr><td><input name='user' autocapitalize='off'><td><input name='host' value='localhost' autocapitalize='off'><td><input type='submit' value='".'Edit'."'>\n";echo"</table>\n","</form>\n";}elseif(isset($_GET["sql"])){if(!$l&&$_POST["export"]){save_settings(array("output"=>$_POST["output"],"format"=>$_POST["format"]),"adminer_import");dump_headers("sql");adminer()->dumpTable("","");adminer()->dumpData("","table",$_POST["query"]);adminer()->dumpFooter();exit;}restart_session();$Hd=&get_session("queries");$Gd=&$Hd[DB];if(!$l&&$_POST["clear"]){$Gd=array();redirect(remove_from_uri("history"));}stop_session();page_header((isset($_GET["import"])?'Import':'SQL command'),$l);$Fe='--'.(JUSH=='sql'?' ':'');if(!$l&&$_POST){$q=false;if(!isset($_GET["import"]))$H=$_POST["query"];elseif($_POST["webfile"]){$Oh=adminer()->importServerPath();$q=@fopen((file_exists($Oh)?$Oh:"compress.zlib://$Oh.gz"),"rb");$H=($q?fread($q,1e6):false);}else$H=get_file("sql_file",true,";");if(is_string($H)){if(function_exists('memory_get_usage')&&($We=ini_bytes("memory_limit"))!="-1")@ini_set("memory_limit",max($We,strval(2*strlen($H)+memory_get_usage()+8e6)));if($H!=""&&strlen($H)<1e6){$Jg=$H.(preg_match("~;[ \t\r\n]*\$~",$H)?"":";");if(!$Gd||first(end($Gd))!=$Jg){restart_session();$Gd[]=array($Jg,time());set_session("queries",$Hd);stop_session();}}$Lh="(?:\\s|/\\*[\s\S]*?\\*/|(?:#|$Fe)[^\n]*\n?|--\r?\n)";$Ub=";";$D=0;$uc=true;$g=connect();if($g&&DB!=""){$g->select_db(DB);if($_GET["ns"]!="")set_schema($_GET["ns"],$g);}$lb=0;$Bc=array();$gg='[\'"'.(JUSH=="sql"?'`#':(JUSH=="sqlite"?'`[':(JUSH=="mssql"?'[':''))).']|/\*|'.$Fe.'|$'.(JUSH=="pgsql"?'|\$[^$]*\$':'');$Fi=microtime(true);$ma=get_settings("adminer_import");$lc=adminer()->dumpFormat();unset($lc["sql"]);while($H!=""){if(!$D&&preg_match("~^$Lh*+DELIMITER\\s+(\\S+)~i",$H,$B)){$Ub=preg_quote($B[1]);$H=substr($H,strlen($B[0]));}elseif(!$D&&JUSH=='pgsql'&&preg_match("~^($Lh*+COPY\\s+)[^;]+\\s+FROM\\s+stdin;~i",$H,$B)){$Ub="\n\\\\\\.\r?\n";$D=strlen($B[0]);}else{preg_match("($Ub\\s*|$gg)",$H,$B,PREG_OFFSET_CAPTURE,$D);list($kd,$vg)=$B[0];if(!$kd&&$q&&!feof($q))$H
.=fread($q,1e5);else{if(!$kd&&rtrim($H)=="")break;$D=$vg+strlen($kd);if($kd&&!preg_match("(^$Ub)",$kd)){$Qa=driver()->hasCStyleEscapes()||(JUSH=="pgsql"&&($vg>0&&strtolower($H[$vg-1])=="e"));$og=($kd=='/*'?'\*/':($kd=='['?']':(preg_match("~^$Fe|^#~",$kd)?"\n":preg_quote($kd).($Qa?'|\\\\.':''))));while(preg_match("($og|\$)s",$H,$B,PREG_OFFSET_CAPTURE,$D)){$kh=$B[0][0];if(!$kh&&$q&&!feof($q))$H
.=fread($q,1e5);else{$D=$B[0][1]+strlen($kh);if(!$kh||$kh[0]!="\\")break;}}}else{$uc=false;$Jg=substr($H,0,$vg+($Ub[0]=="\n"?3:0));$lb++;$Cg="<pre id='sql-$lb'><code class='jush-".JUSH."'>".adminer()->sqlCommandQuery($Jg)."</code></pre>\n";if(JUSH=="sqlite"&&preg_match("~^$Lh*+ATTACH\\b~i",$Jg,$B)){echo$Cg,"<p class='error'>".'ATTACH queries are not supported.'."\n";$Bc[]=" <a href='#sql-$lb'>$lb</a>";if($_POST["error_stops"])break;}else{if(!$_POST["only_errors"]){echo$Cg;ob_flush();flush();}$Th=microtime(true);if(connection()->multi_query($Jg)&&$g&&preg_match("~^$Lh*+USE\\b~i",$Jg))$g->query($Jg);do{$I=connection()->store_result();if(connection()->error){echo($_POST["only_errors"]?$Cg:""),"<p class='error'>".'Error in query'.(connection()->errno?" (".connection()->errno.")":"").": ".error()."\n";$Bc[]=" <a href='#sql-$lb'>$lb</a>";if($_POST["error_stops"])break
2;}else{$vi=" <span class='time'>(".format_time($Th).")</span>".(strlen($Jg)<1000?" <a href='".h(ME)."sql=".urlencode(trim($Jg))."'>".'Edit'."</a>":"");$oa=connection()->affected_rows;$uj=($_POST["only_errors"]?"":driver()->warnings());$vj="warnings-$lb";if($uj)$vi
.=", <a href='#$vj'>".'Warnings'."</a>".script("qsl('a').onclick = partial(toggle, '$vj');","");$Jc=null;$Rf=null;$Kc="explain-$lb";if(is_object($I)){$z=$_POST["limit"];$Rf=print_select_result($I,$g,array(),$z);if(!$_POST["only_errors"]){echo"<form action='' method='post'>\n";$tf=$I->num_rows;echo"<p class='sql-footer'>".($tf?($z&&$tf>$z?sprintf('%d / ',$z):"").lang_format(array('%d row','%d rows'),$tf):""),$vi;if($g&&preg_match("~^($Lh|\\()*+SELECT\\b~i",$Jg)&&($Jc=explain($g,$Jg)))echo", <a href='#$Kc'>Explain</a>".script("qsl('a').onclick = partial(toggle, '$Kc');","");$t="export-$lb";echo", <a href='#$t'>".'Export'."</a>".script("qsl('a').onclick = partial(toggle, '$t');","")."<span id='$t' class='hidden'>: ".html_select("output",adminer()->dumpOutput(),$ma["output"])." ".html_select("format",$lc,$ma["format"]).input_hidden("query",$Jg)."<input type='submit' name='export' value='".'Export'."'>".input_token()."</span>\n"."</form>\n";}}else{if(preg_match("~^$Lh*+(CREATE|DROP|ALTER)$Lh++(DATABASE|SCHEMA)\\b~i",$Jg)){restart_session();set_session("dbs",null);stop_session();}if(!$_POST["only_errors"])echo"<p class='message' title='".h(connection()->info)."'>".lang_format(array('Query executed OK, %d row affected.','Query executed OK, %d rows affected.'),$oa)."$vi\n";}echo($uj?"<div id='$vj' class='hidden'>\n$uj</div>\n":"");if($Jc){echo"<div id='$Kc' class='hidden explain'>\n";print_select_result($Jc,$g,$Rf);echo"</div>\n";}}$Th=microtime(true);}while(connection()->next_result());}$H=substr($H,$D);$D=0;}}}}if($uc)echo"<p class='message'>".'No commands to execute.'."\n";elseif($_POST["only_errors"])echo"<p class='message'>".lang_format(array('%d query executed OK.','%d queries executed OK.'),$lb-count($Bc))," <span class='time'>(".format_time($Fi).")</span>\n";elseif($Bc&&$lb>1)echo"<p class='error'>".'Error in query'.": ".implode("",$Bc)."\n";}else
echo"<p class='error'>".upload_error($H)."\n";}echo'
<form action="" method="post" enctype="multipart/form-data" id="form">
';$Hc="<input type='submit' value='".'Execute'."' title='Ctrl+Enter'>";if(!isset($_GET["import"])){$Jg=$_GET["sql"];if($_POST)$Jg=$_POST["query"];elseif($_GET["history"]=="all")$Jg=$Gd;elseif($_GET["history"]!="")$Jg=idx($Gd[$_GET["history"]],0);echo"<p>";textarea("query",$Jg,20);echo
script(($_POST?"":"qs('textarea').focus();\n")."qs('#form').onsubmit = partial(sqlSubmit, qs('#form'), '".js_escape(remove_from_uri("sql|limit|error_stops|only_errors|history"))."');"),"<p>";adminer()->sqlPrintAfter();echo"$Hc\n",'Limit rows'.": <input type='number' name='limit' class='size' value='".h($_POST?$_POST["limit"]:$_GET["limit"])."'>\n";}else{echo"<fieldset><legend>".'File upload'."</legend><div>";$xd=(extension_loaded("zlib")?"[.gz]":"");echo(ini_bool("file_uploads")?"SQL$xd (&lt; ".ini_get("upload_max_filesize")."B): <input type='file' name='sql_file[]' multiple>\n$Hc":'File uploads are disabled.'),"</div></fieldset>\n";$Rd=adminer()->importServerPath();if($Rd)echo"<fieldset><legend>".'From server'."</legend><div>",sprintf('Webserver file %s',"<code>".h($Rd)."$xd</code>"),' <input type="submit" name="webfile" value="'.'Run file'.'">',"</div></fieldset>\n";echo"<p>";}echo
checkbox("error_stops",1,($_POST?$_POST["error_stops"]:isset($_GET["import"])||$_GET["error_stops"]),'Stop on error')."\n",checkbox("only_errors",1,($_POST?$_POST["only_errors"]:isset($_GET["import"])||$_GET["only_errors"]),'Show only errors')."\n",input_token();if(!isset($_GET["import"])&&$Gd){print_fieldset("history",'History',$_GET["history"]!="");for($X=end($Gd);$X;$X=prev($Gd)){$x=key($Gd);list($Jg,$vi,$pc)=$X;echo'<a href="'.h(ME."sql=&history=$x").'">'.'Edit'."</a>"." <span class='time' title='".@date('Y-m-d',$vi)."'>".@date("H:i:s",$vi)."</span>"." <code class='jush-".JUSH."'>".shorten_utf8(ltrim(str_replace("\n"," ",str_replace("\r","",preg_replace("~^(#|$Fe).*~m",'',$Jg)))),80,"</code>").($pc?" <span class='time'>($pc)</span>":"")."<br>\n";}echo"<input type='submit' name='clear' value='".'Clear'."'>\n","<a href='".h(ME."sql=&history=all")."'>".'Edit all'."</a>\n","</div></fieldset>\n";}echo'</form>
';}elseif(isset($_GET["edit"])){$a=$_GET["edit"];$n=fields($a);$Z=(isset($_GET["select"])?($_POST["check"]&&count($_POST["check"])==1?where_check($_POST["check"][0],$n):""):where($_GET,$n));$bj=(isset($_GET["select"])?$_POST["edit"]:$Z);foreach($n
as$C=>$m){if(!isset($m["privileges"][$bj?"update":"insert"])||adminer()->fieldName($m)==""||$m["generated"])unset($n[$C]);}if($_POST&&!$l&&!isset($_GET["select"])){$A=$_POST["referer"];if($_POST["insert"])$A=($bj?null:$_SERVER["REQUEST_URI"]);elseif(!preg_match('~^.+&select=.+$~',$A))$A=ME."select=".urlencode($a);$w=indexes($a);$Wi=unique_array($_GET["where"],$w);$Mg="\nWHERE $Z";if(isset($_POST["delete"]))queries_redirect($A,'Item has been deleted.',driver()->delete($a,$Mg,$Wi?0:1));else{$O=array();foreach($n
as$C=>$m){$X=process_input($m);if($X!==false&&$X!==null)$O[idf_escape($C)]=$X;}if($bj){if(!$O)redirect($A);queries_redirect($A,'Item has been updated.',driver()->update($a,$O,$Mg,$Wi?0:1));if(is_ajax()){page_headers();page_messages($l);exit;}}else{$I=driver()->insert($a,$O);$ze=($I?last_id($I):0);queries_redirect($A,sprintf('Item%s has been inserted.',($ze?" $ze":"")),$I);}}}$K=null;if($_POST["save"])$K=(array)$_POST["fields"];elseif($Z){$M=array();foreach($n
as$C=>$m){if(isset($m["privileges"]["select"])){$wa=($_POST["clone"]&&$m["auto_increment"]?"''":convert_field($m));$M[]=($wa?"$wa AS ":"").idf_escape($C);}}$K=array();if(!support("table"))$M=array("*");if($M){$I=driver()->select($a,$M,array($Z),$M,array(),(isset($_GET["select"])?2:1));if(!$I)$l=error();else{$K=$I->fetch_assoc();if(!$K)$K=false;}if(isset($_GET["select"])&&(!$K||$I->fetch_assoc()))$K=null;}}if(!support("table")&&!$n){if(!$Z){$I=driver()->select($a,array("*"),array(),array("*"));$K=($I?$I->fetch_assoc():false);if(!$K)$K=array(driver()->primary=>"");}if($K){foreach($K
as$x=>$X){if(!$Z)$K[$x]=null;$n[$x]=array("field"=>$x,"null"=>($x!=driver()->primary),"auto_increment"=>($x==driver()->primary));}}}edit_form($a,$n,$K,$bj,$l);}elseif(isset($_GET["create"])){$a=$_GET["create"];$ig=array();foreach(array('HASH','LINEAR HASH','KEY','LINEAR KEY','RANGE','LIST')as$x)$ig[$x]=$x;$Sg=referencable_primary($a);$id=array();foreach($Sg
as$fi=>$m)$id[str_replace("`","``",$fi)."`".str_replace("`","``",$m["field"])]=$fi;$Uf=array();$S=array();if($a!=""){$Uf=fields($a);$S=table_status1($a);if(count($S)<2)$l='No tables.';}$K=$_POST;$K["fields"]=(array)$K["fields"];if($K["auto_increment_col"])$K["fields"][$K["auto_increment_col"]]["auto_increment"]=true;if($_POST)save_settings(array("comments"=>$_POST["comments"],"defaults"=>$_POST["defaults"]));if($_POST&&!process_fields($K["fields"])&&!$l){if($_POST["drop"])queries_redirect(substr(ME,0,-1),'Table has been dropped.',drop_tables(array($a)));else{$n=array();$sa=array();$fj=false;$gd=array();$Tf=reset($Uf);$qa=" FIRST";foreach($K["fields"]as$x=>$m){$p=$id[$m["type"]];$Qi=($p!==null?$Sg[$p]:$m);if($m["field"]!=""){if(!$m["generated"])$m["default"]=null;$Hg=process_field($m,$Qi);$sa[]=array($m["orig"],$Hg,$qa);if(!$Tf||$Hg!==process_field($Tf,$Tf)){$n[]=array($m["orig"],$Hg,$qa);if($m["orig"]!=""||$qa)$fj=true;}if($p!==null)$gd[idf_escape($m["field"])]=($a!=""&&JUSH!="sqlite"?"ADD":" ").format_foreign_key(array('table'=>$id[$m["type"]],'source'=>array($m["field"]),'target'=>array($Qi["field"]),'on_delete'=>$m["on_delete"],));$qa=" AFTER ".idf_escape($m["field"]);}elseif($m["orig"]!=""){$fj=true;$n[]=array($m["orig"]);}if($m["orig"]!=""){$Tf=next($Uf);if(!$Tf)$qa="";}}$kg="";if(support("partitioning")){if(isset($ig[$K["partition_by"]])){$fg=array();foreach($K
as$x=>$X){if(preg_match('~^partition~',$x))$fg[$x]=$X;}foreach($fg["partition_names"]as$x=>$C){if($C==""){unset($fg["partition_names"][$x]);unset($fg["partition_values"][$x]);}}if($fg!=get_partitions_info($a)){$lg=array();if($fg["partition_by"]=='RANGE'||$fg["partition_by"]=='LIST'){foreach($fg["partition_names"]as$x=>$C){$Y=$fg["partition_values"][$x];$lg[]="\n  PARTITION ".idf_escape($C)." VALUES ".($fg["partition_by"]=='RANGE'?"LESS THAN":"IN").($Y!=""?" ($Y)":" MAXVALUE");}}$kg
.="\nPARTITION BY $fg[partition_by]($fg[partition])";if($lg)$kg
.=" (".implode(",",$lg)."\n)";elseif($fg["partitions"])$kg
.=" PARTITIONS ".(+$fg["partitions"]);}}elseif(preg_match("~partitioned~",$S["Create_options"]))$kg
.="\nREMOVE PARTITIONING";}$Xe='Table has been altered.';if($a==""){cookie("adminer_engine",$K["Engine"]);$Xe='Table has been created.';}$C=trim($K["name"]);queries_redirect(ME.(support("table")?"table=":"select=").urlencode($C),$Xe,alter_table($a,$C,(JUSH=="sqlite"&&($fj||$gd)?$sa:$n),$gd,($K["Comment"]!=$S["Comment"]?$K["Comment"]:null),($K["Engine"]&&$K["Engine"]!=$S["Engine"]?$K["Engine"]:""),($K["Collation"]&&$K["Collation"]!=$S["Collation"]?$K["Collation"]:""),($K["Auto_increment"]!=""?number($K["Auto_increment"]):""),$kg));}}page_header(($a!=""?'Alter table':'Create table'),$l,array("table"=>$a),h($a));if(!$_POST){$Si=driver()->types();$K=array("Engine"=>$_COOKIE["adminer_engine"],"fields"=>array(array("field"=>"","type"=>(isset($Si["int"])?"int":(isset($Si["integer"])?"integer":"")),"on_update"=>"")),"partition_names"=>array(""),);if($a!=""){$K=$S;$K["name"]=$a;$K["fields"]=array();if(!$_GET["auto_increment"])$K["Auto_increment"]="";foreach($Uf
as$m){$m["generated"]=$m["generated"]?:(isset($m["default"])?"DEFAULT":"");$K["fields"][]=$m;}if(support("partitioning")){$K+=get_partitions_info($a);$K["partition_names"][]="";$K["partition_values"][]="";}}}$hb=collations();if(is_array(reset($hb)))$hb=call_user_func_array('array_merge',array_values($hb));$wc=driver()->engines();foreach($wc
as$vc){if(!strcasecmp($vc,$K["Engine"])){$K["Engine"]=$vc;break;}}echo'
<form action="" method="post" id="form">
<p>
';if(support("columns")||$a==""){echo'Table name'.": <input name='name'".($a==""&&!$_POST?" autofocus":"")." data-maxlength='64' value='".h($K["name"])."' autocapitalize='off'>\n",($wc?html_select("Engine",array(""=>"(".'engine'.")")+$wc,$K["Engine"]).on_help("event.target.value",1).script("qsl('select').onchange = helpClose;")."\n":"");if($hb)echo"<datalist id='collations'>".optionlist($hb)."</datalist>\n",(preg_match("~sqlite|mssql~",JUSH)?"":"<input list='collations' name='Collation' value='".h($K["Collation"])."' placeholder='(".'collation'.")'>\n");echo"<input type='submit' value='".'Save'."'>\n";}if(support("columns")){echo"<div class='scrollable'>\n","<table id='edit-fields' class='nowrap'>\n";edit_fields($K["fields"],$hb,"TABLE",$id);echo"</table>\n",script("editFields();"),"</div>\n<p>\n",'Auto Increment'.": <input type='number' name='Auto_increment' class='size' value='".h($K["Auto_increment"])."'>\n",checkbox("defaults",1,($_POST?$_POST["defaults"]:get_setting("defaults")),'Default values',"columnShow(this.checked, 5)","jsonly");$ob=($_POST?$_POST["comments"]:get_setting("comments"));echo(support("comment")?checkbox("comments",1,$ob,'Comment',"editingCommentsClick(this, true);","jsonly").' '.(preg_match('~\n~',$K["Comment"])?"<textarea name='Comment' rows='2' cols='20'".($ob?"":" class='hidden'").">".h($K["Comment"])."</textarea>":'<input name="Comment" value="'.h($K["Comment"]).'" data-maxlength="'.(min_version(5.5)?2048:60).'"'.($ob?"":" class='hidden'").'>'):''),'<p>
<input type="submit" value="Save">
';}echo'
';if($a!="")echo'<input type="submit" name="drop" value="Drop">',confirm(sprintf('Drop %s?',$a));if(support("partitioning")){$jg=preg_match('~RANGE|LIST~',$K["partition_by"]);print_fieldset("partition",'Partition by',$K["partition_by"]);echo"<p>".html_select("partition_by",array(""=>"")+$ig,$K["partition_by"]).on_help("event.target.value.replace(/./, 'PARTITION BY \$&')",1).script("qsl('select').onchange = partitionByChange;"),"(<input name='partition' value='".h($K["partition"])."'>)\n",'Partitions'.": <input type='number' name='partitions' class='size".($jg||!$K["partition_by"]?" hidden":"")."' value='".h($K["partitions"])."'>\n","<table id='partition-table'".($jg?"":" class='hidden'").">\n","<thead><tr><th>".'Partition name'."<th>".'Values'."</thead>\n";foreach($K["partition_names"]as$x=>$X)echo'<tr>','<td><input name="partition_names[]" value="'.h($X).'" autocapitalize="off">',($x==count($K["partition_names"])-1?script("qsl('input').oninput = partitionNameChange;"):''),'<td><input name="partition_values[]" value="'.h(idx($K["partition_values"],$x)).'">';echo"</table>\n</div></fieldset>\n";}echo
input_token(),'</form>
';}elseif(isset($_GET["indexes"])){$a=$_GET["indexes"];$Wd=array("PRIMARY","UNIQUE","INDEX");$S=table_status1($a,true);if(preg_match('~MyISAM|M?aria'.(min_version(5.6,'10.0.5')?'|InnoDB':'').'~i',$S["Engine"]))$Wd[]="FULLTEXT";if(preg_match('~MyISAM|M?aria'.(min_version(5.7,'10.2.2')?'|InnoDB':'').'~i',$S["Engine"]))$Wd[]="SPATIAL";$w=indexes($a);$G=array();if(JUSH=="mongo"){$G=$w["_id_"];unset($Wd[0]);unset($w["_id_"]);}$K=$_POST;if($K)save_settings(array("index_options"=>$K["options"]));if($_POST&&!$l&&!$_POST["add"]&&!$_POST["drop_col"]){$b=array();foreach($K["indexes"]as$v){$C=$v["name"];if(in_array($v["type"],$Wd)){$e=array();$De=array();$Xb=array();$O=array();ksort($v["columns"]);foreach($v["columns"]as$x=>$d){if($d!=""){$y=idx($v["lengths"],$x);$Vb=idx($v["descs"],$x);$O[]=idf_escape($d).($y?"(".(+$y).")":"").($Vb?" DESC":"");$e[]=$d;$De[]=($y?:null);$Xb[]=$Vb;}}$Ic=$w[$C];if($Ic){ksort($Ic["columns"]);ksort($Ic["lengths"]);ksort($Ic["descs"]);if($v["type"]==$Ic["type"]&&array_values($Ic["columns"])===$e&&(!$Ic["lengths"]||array_values($Ic["lengths"])===$De)&&array_values($Ic["descs"])===$Xb){unset($w[$C]);continue;}}if($e)$b[]=array($v["type"],$C,$O);}}foreach($w
as$C=>$Ic)$b[]=array($Ic["type"],$C,"DROP");if(!$b)redirect(ME."table=".urlencode($a));queries_redirect(ME."table=".urlencode($a),'Indexes have been altered.',alter_indexes($a,$b));}page_header('Indexes',$l,array("table"=>$a),h($a));$n=array_keys(fields($a));if($_POST["add"]){foreach($K["indexes"]as$x=>$v){if($v["columns"][count($v["columns"])]!="")$K["indexes"][$x]["columns"][]="";}$v=end($K["indexes"]);if($v["type"]||array_filter($v["columns"],'strlen'))$K["indexes"][]=array("columns"=>array(1=>""));}if(!$K){foreach($w
as$x=>$v){$w[$x]["name"]=$x;$w[$x]["columns"][]="";}$w[]=array("columns"=>array(1=>""));$K["indexes"]=$w;}$De=(JUSH=="sql"||JUSH=="mssql");$Fh=($_POST?$_POST["options"]:get_setting("index_options"));echo'
<form action="" method="post">
<div class="scrollable">
<table class="nowrap">
<thead><tr>
<th id="label-type">Index Type
<th><input type="submit" class="wayoff">','Columns'.($De?"<span class='idxopts".($Fh?"":" hidden")."'> (".'length'.")</span>":"");if($De||support("descidx"))echo
checkbox("options",1,$Fh,'Options',"indexOptionsShow(this.checked)","jsonly")."\n";echo'<th id="label-name">Name
<th><noscript>',icon("plus","add[0]","+",'Add next'),'</noscript>
</thead>
';if($G){echo"<tr><td>PRIMARY<td>";foreach($G["columns"]as$x=>$d)echo
select_input(" disabled",$n,$d),"<label><input disabled type='checkbox'>".'descending'."</label> ";echo"<td><td>\n";}$oe=1;foreach($K["indexes"]as$v){if(!$_POST["drop_col"]||$oe!=key($_POST["drop_col"])){echo"<tr><td>".html_select("indexes[$oe][type]",array(-1=>"")+$Wd,$v["type"],($oe==count($K["indexes"])?"indexesAddRow.call(this);":""),"label-type"),"<td>";ksort($v["columns"]);$s=1;foreach($v["columns"]as$x=>$d){echo"<span>".select_input(" name='indexes[$oe][columns][$s]' title='".'Column'."'",($n?array_combine($n,$n):$n),$d,"partial(".($s==count($v["columns"])?"indexesAddColumn":"indexesChangeColumn").", '".js_escape(JUSH=="sql"?"":$_GET["indexes"]."_")."')"),"<span class='idxopts".($Fh?"":" hidden")."'>",($De?"<input type='number' name='indexes[$oe][lengths][$s]' class='size' value='".h(idx($v["lengths"],$x))."' title='".'Length'."'>":""),(support("descidx")?checkbox("indexes[$oe][descs][$s]",1,idx($v["descs"],$x),'descending'):""),"</span> </span>";$s++;}echo"<td><input name='indexes[$oe][name]' value='".h($v["name"])."' autocapitalize='off' aria-labelledby='label-name'>\n","<td>".icon("cross","drop_col[$oe]","x",'Remove').script("qsl('button').onclick = partial(editingRemoveRow, 'indexes\$1[type]');");}$oe++;}echo'</table>
</div>
<p>
<input type="submit" value="Save">
',input_token(),'</form>
';}elseif(isset($_GET["database"])){$K=$_POST;if($_POST&&!$l&&!$_POST["add"]){$C=trim($K["name"]);if($_POST["drop"]){$_GET["db"]="";queries_redirect(remove_from_uri("db|database"),'Database has been dropped.',drop_databases(array(DB)));}elseif(DB!==$C){if(DB!=""){$_GET["db"]=$C;queries_redirect(preg_replace('~\bdb=[^&]*&~','',ME)."db=".urlencode($C),'Database has been renamed.',rename_database($C,$K["collation"]));}else{$i=explode("\n",str_replace("\r","",$C));$Yh=true;$ye="";foreach($i
as$j){if(count($i)==1||$j!=""){if(!create_database($j,$K["collation"]))$Yh=false;$ye=$j;}}restart_session();set_session("dbs",null);queries_redirect(ME."db=".urlencode($ye),'Database has been created.',$Yh);}}else{if(!$K["collation"])redirect(substr(ME,0,-1));query_redirect("ALTER DATABASE ".idf_escape($C).(preg_match('~^[a-z0-9_]+$~i',$K["collation"])?" COLLATE $K[collation]":""),substr(ME,0,-1),'Database has been altered.');}}page_header(DB!=""?'Alter database':'Create database',$l,array(),h(DB));$hb=collations();$C=DB;if($_POST)$C=$K["name"];elseif(DB!="")$K["collation"]=db_collation(DB,$hb);elseif(JUSH=="sql"){foreach(get_vals("SHOW GRANTS")as$rd){if(preg_match('~ ON (`(([^\\\\`]|``|\\\\.)*)%`\.\*)?~',$rd,$B)&&$B[1]){$C=stripcslashes(idf_unescape("`$B[2]`"));break;}}}echo'
<form action="" method="post">
<p>
',($_POST["add"]||strpos($C,"\n")?'<textarea autofocus name="name" rows="10" cols="40">'.h($C).'</textarea><br>':'<input name="name" autofocus value="'.h($C).'" data-maxlength="64" autocapitalize="off">')."\n".($hb?html_select("collation",array(""=>"(".'collation'.")")+$hb,$K["collation"]).doc_link(array('sql'=>"charset-charsets.html",'mariadb'=>"supported-character-sets-and-collations/",'mssql'=>"relational-databases/system-functions/sys-fn-helpcollations-transact-sql",)):""),'<input type="submit" value="Save">
';if(DB!="")echo"<input type='submit' name='drop' value='".'Drop'."'>".confirm(sprintf('Drop %s?',DB))."\n";elseif(!$_POST["add"]&&$_GET["db"]=="")echo
icon("plus","add[0]","+",'Add next')."\n";echo
input_token(),'</form>
';}elseif(isset($_GET["scheme"])){$K=$_POST;if($_POST&&!$l){$_=preg_replace('~ns=[^&]*&~','',ME)."ns=";if($_POST["drop"])query_redirect("DROP SCHEMA ".idf_escape($_GET["ns"]),$_,'Schema has been dropped.');else{$C=trim($K["name"]);$_
.=urlencode($C);if($_GET["ns"]=="")query_redirect("CREATE SCHEMA ".idf_escape($C),$_,'Schema has been created.');elseif($_GET["ns"]!=$C)query_redirect("ALTER SCHEMA ".idf_escape($_GET["ns"])." RENAME TO ".idf_escape($C),$_,'Schema has been altered.');else
redirect($_);}}page_header($_GET["ns"]!=""?'Alter schema':'Create schema',$l);if(!$K)$K["name"]=$_GET["ns"];echo'
<form action="" method="post">
<p><input name="name" autofocus value="',h($K["name"]),'" autocapitalize="off">
<input type="submit" value="Save">
';if($_GET["ns"]!="")echo"<input type='submit' name='drop' value='".'Drop'."'>".confirm(sprintf('Drop %s?',$_GET["ns"]))."\n";echo
input_token(),'</form>
';}elseif(isset($_GET["call"])){$ba=($_GET["name"]?:$_GET["call"]);page_header('Call'.": ".h($ba),$l);$gh=routine($_GET["call"],(isset($_GET["callf"])?"FUNCTION":"PROCEDURE"));$Sd=array();$Zf=array();foreach($gh["fields"]as$s=>$m){if(substr($m["inout"],-3)=="OUT")$Zf[$s]="@".idf_escape($m["field"])." AS ".idf_escape($m["field"]);if(!$m["inout"]||substr($m["inout"],0,2)=="IN")$Sd[]=$s;}if(!$l&&$_POST){$Ra=array();foreach($gh["fields"]as$x=>$m){$X="";if(in_array($x,$Sd)){$X=process_input($m);if($X===false)$X="''";if(isset($Zf[$x]))connection()->query("SET @".idf_escape($m["field"])." = $X");}$Ra[]=(isset($Zf[$x])?"@".idf_escape($m["field"]):$X);}$H=(isset($_GET["callf"])?"SELECT":"CALL")." ".table($ba)."(".implode(", ",$Ra).")";$Th=microtime(true);$I=connection()->multi_query($H);$oa=connection()->affected_rows;echo
adminer()->selectQuery($H,$Th,!$I);if(!$I)echo"<p class='error'>".error()."\n";else{$g=connect();if($g)$g->select_db(DB);do{$I=connection()->store_result();if(is_object($I))print_select_result($I,$g);else
echo"<p class='message'>".lang_format(array('Routine has been called, %d row affected.','Routine has been called, %d rows affected.'),$oa)." <span class='time'>".@date("H:i:s")."</span>\n";}while(connection()->next_result());if($Zf)print_select_result(connection()->query("SELECT ".implode(", ",$Zf)));}}echo'
<form action="" method="post">
';if($Sd){echo"<table class='layout'>\n";foreach($Sd
as$x){$m=$gh["fields"][$x];$C=$m["field"];echo"<tr><th>".adminer()->fieldName($m);$Y=idx($_POST["fields"],$C);if($Y!=""){if($m["type"]=="set")$Y=implode(",",$Y);}input($m,$Y,idx($_POST["function"],$C,""));echo"\n";}echo"</table>\n";}echo'<p>
<input type="submit" value="Call">
',input_token(),'</form>

<pre>
';function
pre_tr($kh){return
preg_replace('~^~m','<tr>',preg_replace('~\|~','<td>',preg_replace('~\|$~m',"",rtrim($kh))));}$R='(\+--[-+]+\+\n)';$K='(\| .* \|\n)';echo
preg_replace_callback("~^$R?$K$R?($K*)$R?~m",function($B){$Zc=pre_tr($B[2]);return"<table>\n".($B[1]?"<thead>$Zc</thead>\n":$Zc).pre_tr($B[4])."\n</table>";},preg_replace('~(\n(    -|mysql)&gt; )(.+)~',"\\1<code class='jush-sql'>\\3</code>",preg_replace('~(.+)\n---+\n~',"<b>\\1</b>\n",h($gh['comment']))));echo'</pre>
';}elseif(isset($_GET["foreign"])){$a=$_GET["foreign"];$C=$_GET["name"];$K=$_POST;if($_POST&&!$l&&!$_POST["add"]&&!$_POST["change"]&&!$_POST["change-js"]){if(!$_POST["drop"]){$K["source"]=array_filter($K["source"],'strlen');ksort($K["source"]);$oi=array();foreach($K["source"]as$x=>$X)$oi[$x]=$K["target"][$x];$K["target"]=$oi;}if(JUSH=="sqlite")$I=recreate_table($a,$a,array(),array(),array(" $C"=>($K["drop"]?"":" ".format_foreign_key($K))));else{$b="ALTER TABLE ".table($a);$I=($C==""||queries("$b DROP ".(JUSH=="sql"?"FOREIGN KEY ":"CONSTRAINT ").idf_escape($C)));if(!$K["drop"])$I=queries("$b ADD".format_foreign_key($K));}queries_redirect(ME."table=".urlencode($a),($K["drop"]?'Foreign key has been dropped.':($C!=""?'Foreign key has been altered.':'Foreign key has been created.')),$I);if(!$K["drop"])$l='Source and target columns must have the same data type, there must be an index on the target columns and referenced data must exist.';}page_header('Foreign key',$l,array("table"=>$a),h($a));if($_POST){ksort($K["source"]);if($_POST["add"])$K["source"][]="";elseif($_POST["change"]||$_POST["change-js"])$K["target"]=array();}elseif($C!=""){$id=foreign_keys($a);$K=$id[$C];$K["source"][]="";}else{$K["table"]=$a;$K["source"]=array("");}echo'
<form action="" method="post">
';$Kh=array_keys(fields($a));if($K["db"]!="")connection()->select_db($K["db"]);if($K["ns"]!=""){$Vf=get_schema();set_schema($K["ns"]);}$Rg=array_keys(array_filter(table_status('',true),'Adminer\fk_support'));$oi=array_keys(fields(in_array($K["table"],$Rg)?$K["table"]:reset($Rg)));$Ff="this.form['change-js'].value = '1'; this.form.submit();";echo"<p><label>".'Target table'.": ".html_select("table",$Rg,$K["table"],$Ff)."</label>\n";if(support("scheme")){$nh=array_filter(adminer()->schemas(),function($mh){return!preg_match('~^information_schema$~i',$mh);});echo"<label>".'Schema'.": ".html_select("ns",$nh,$K["ns"]!=""?$K["ns"]:$_GET["ns"],$Ff)."</label>";if($K["ns"]!="")set_schema($Vf);}elseif(JUSH!="sqlite"){$Ob=array();foreach(adminer()->databases()as$j){if(!information_schema($j))$Ob[]=$j;}echo"<label>".'DB'.": ".html_select("db",$Ob,$K["db"]!=""?$K["db"]:$_GET["db"],$Ff)."</label>";}echo
input_hidden("change-js"),'<noscript><p><input type="submit" name="change" value="Change"></noscript>
<table>
<thead><tr><th id="label-source">Source<th id="label-target">Target</thead>
';$oe=0;foreach($K["source"]as$x=>$X){echo"<tr>","<td>".html_select("source[".(+$x)."]",array(-1=>"")+$Kh,$X,($oe==count($K["source"])-1?"foreignAddRow.call(this);":""),"label-source"),"<td>".html_select("target[".(+$x)."]",$oi,idx($K["target"],$x),"","label-target");$oe++;}echo'</table>
<p>
<label>ON DELETE: ',html_select("on_delete",array(-1=>"")+explode("|",driver()->onActions),$K["on_delete"]),'</label>
<label>ON UPDATE: ',html_select("on_update",array(-1=>"")+explode("|",driver()->onActions),$K["on_update"]),'</label>
',doc_link(array('sql'=>"innodb-foreign-key-constraints.html",'mariadb'=>"foreign-keys/",'pgsql'=>"sql-createtable.html#SQL-CREATETABLE-REFERENCES",'mssql'=>"t-sql/statements/create-table-transact-sql",'oracle'=>"SQLRF01111",)),'<p>
<input type="submit" value="Save">
<noscript><p><input type="submit" name="add" value="Add column"></noscript>
';if($C!="")echo'<input type="submit" name="drop" value="Drop">',confirm(sprintf('Drop %s?',$C));echo
input_token(),'</form>
';}elseif(isset($_GET["view"])){$a=$_GET["view"];$K=$_POST;$Wf="VIEW";if(JUSH=="pgsql"&&$a!=""){$P=table_status1($a);$Wf=strtoupper($P["Engine"]);}if($_POST&&!$l){$C=trim($K["name"]);$wa=" AS\n$K[select]";$A=ME."table=".urlencode($C);$Xe='View has been altered.';$U=($_POST["materialized"]?"MATERIALIZED VIEW":"VIEW");if(!$_POST["drop"]&&$a==$C&&JUSH!="sqlite"&&$U=="VIEW"&&$Wf=="VIEW")query_redirect((JUSH=="mssql"?"ALTER":"CREATE OR REPLACE")." VIEW ".table($C).$wa,$A,$Xe);else{$qi=$C."_adminer_".uniqid();drop_create("DROP $Wf ".table($a),"CREATE $U ".table($C).$wa,"DROP $U ".table($C),"CREATE $U ".table($qi).$wa,"DROP $U ".table($qi),($_POST["drop"]?substr(ME,0,-1):$A),'View has been dropped.',$Xe,'View has been created.',$a,$C);}}if(!$_POST&&$a!=""){$K=view($a);$K["name"]=$a;$K["materialized"]=($Wf!="VIEW");if(!$l)$l=error();}page_header(($a!=""?'Alter view':'Create view'),$l,array("table"=>$a),h($a));echo'
<form action="" method="post">
<p>Name: <input name="name" value="',h($K["name"]),'" data-maxlength="64" autocapitalize="off">
',(support("materializedview")?" ".checkbox("materialized",1,$K["materialized"],'Materialized view'):""),'<p>';textarea("select",$K["select"]);echo'<p>
<input type="submit" value="Save">
';if($a!="")echo'<input type="submit" name="drop" value="Drop">',confirm(sprintf('Drop %s?',$a));echo
input_token(),'</form>
';}elseif(isset($_GET["event"])){$aa=$_GET["event"];$ge=array("YEAR","QUARTER","MONTH","DAY","HOUR","MINUTE","WEEK","SECOND","YEAR_MONTH","DAY_HOUR","DAY_MINUTE","DAY_SECOND","HOUR_MINUTE","HOUR_SECOND","MINUTE_SECOND");$Uh=array("ENABLED"=>"ENABLE","DISABLED"=>"DISABLE","SLAVESIDE_DISABLED"=>"DISABLE ON SLAVE");$K=$_POST;if($_POST&&!$l){if($_POST["drop"])query_redirect("DROP EVENT ".idf_escape($aa),substr(ME,0,-1),'Event has been dropped.');elseif(in_array($K["INTERVAL_FIELD"],$ge)&&isset($Uh[$K["STATUS"]])){$lh="\nON SCHEDULE ".($K["INTERVAL_VALUE"]?"EVERY ".q($K["INTERVAL_VALUE"])." $K[INTERVAL_FIELD]".($K["STARTS"]?" STARTS ".q($K["STARTS"]):"").($K["ENDS"]?" ENDS ".q($K["ENDS"]):""):"AT ".q($K["STARTS"]))." ON COMPLETION".($K["ON_COMPLETION"]?"":" NOT")." PRESERVE";queries_redirect(substr(ME,0,-1),($aa!=""?'Event has been altered.':'Event has been created.'),queries(($aa!=""?"ALTER EVENT ".idf_escape($aa).$lh.($aa!=$K["EVENT_NAME"]?"\nRENAME TO ".idf_escape($K["EVENT_NAME"]):""):"CREATE EVENT ".idf_escape($K["EVENT_NAME"]).$lh)."\n".$Uh[$K["STATUS"]]." COMMENT ".q($K["EVENT_COMMENT"]).rtrim(" DO\n$K[EVENT_DEFINITION]",";").";"));}}page_header(($aa!=""?'Alter event'.": ".h($aa):'Create event'),$l);if(!$K&&$aa!=""){$L=get_rows("SELECT * FROM information_schema.EVENTS WHERE EVENT_SCHEMA = ".q(DB)." AND EVENT_NAME = ".q($aa));$K=reset($L);}echo'
<form action="" method="post">
<table class="layout">
<tr><th>Name<td><input name="EVENT_NAME" value="',h($K["EVENT_NAME"]),'" data-maxlength="64" autocapitalize="off">
<tr><th title="datetime">Start<td><input name="STARTS" value="',h("$K[EXECUTE_AT]$K[STARTS]"),'">
<tr><th title="datetime">End<td><input name="ENDS" value="',h($K["ENDS"]),'">
<tr><th>Every<td><input type="number" name="INTERVAL_VALUE" value="',h($K["INTERVAL_VALUE"]),'" class="size"> ',html_select("INTERVAL_FIELD",$ge,$K["INTERVAL_FIELD"]),'<tr><th>Status<td>',html_select("STATUS",$Uh,$K["STATUS"]),'<tr><th>Comment<td><input name="EVENT_COMMENT" value="',h($K["EVENT_COMMENT"]),'" data-maxlength="64">
<tr><th><td>',checkbox("ON_COMPLETION","PRESERVE",$K["ON_COMPLETION"]=="PRESERVE",'On completion preserve'),'</table>
<p>';textarea("EVENT_DEFINITION",$K["EVENT_DEFINITION"]);echo'<p>
<input type="submit" value="Save">
';if($aa!="")echo'<input type="submit" name="drop" value="Drop">',confirm(sprintf('Drop %s?',$aa));echo
input_token(),'</form>
';}elseif(isset($_GET["procedure"])){$ba=($_GET["name"]?:$_GET["procedure"]);$gh=(isset($_GET["function"])?"FUNCTION":"PROCEDURE");$K=$_POST;$K["fields"]=(array)$K["fields"];if($_POST&&!process_fields($K["fields"])&&!$l){$Sf=routine($_GET["procedure"],$gh);$qi="$K[name]_adminer_".uniqid();foreach($K["fields"]as$x=>$m){if($m["field"]=="")unset($K["fields"][$x]);}drop_create("DROP $gh ".routine_id($ba,$Sf),create_routine($gh,$K),"DROP $gh ".routine_id($K["name"],$K),create_routine($gh,array("name"=>$qi)+$K),"DROP $gh ".routine_id($qi,$K),substr(ME,0,-1),'Routine has been dropped.','Routine has been altered.','Routine has been created.',$ba,$K["name"]);}page_header(($ba!=""?(isset($_GET["function"])?'Alter function':'Alter procedure').": ".h($ba):(isset($_GET["function"])?'Create function':'Create procedure')),$l);if(!$_POST){if($ba=="")$K["language"]="sql";else{$K=routine($_GET["procedure"],$gh);$K["name"]=$ba;}}$hb=get_vals("SHOW CHARACTER SET");sort($hb);$hh=routine_languages();echo($hb?"<datalist id='collations'>".optionlist($hb)."</datalist>":""),'
<form action="" method="post" id="form">
<p>Name: <input name="name" value="',h($K["name"]),'" data-maxlength="64" autocapitalize="off">
',($hh?"<label>".'Language'.": ".html_select("language",$hh,$K["language"])."</label>\n":""),'<input type="submit" value="Save">
<div class="scrollable">
<table class="nowrap">
';edit_fields($K["fields"],$hb,$gh);if(isset($_GET["function"])){echo"<tr><td>".'Return type';edit_type("returns",$K["returns"],$hb,array(),(JUSH=="pgsql"?array("void","trigger"):array()));}echo'</table>
',script("editFields();"),'</div>
<p>';textarea("definition",$K["definition"]);echo'<p>
<input type="submit" value="Save">
';if($ba!="")echo'<input type="submit" name="drop" value="Drop">',confirm(sprintf('Drop %s?',$ba));echo
input_token(),'</form>
';}elseif(isset($_GET["sequence"])){$da=$_GET["sequence"];$K=$_POST;if($_POST&&!$l){$_=substr(ME,0,-1);$C=trim($K["name"]);if($_POST["drop"])query_redirect("DROP SEQUENCE ".idf_escape($da),$_,'Sequence has been dropped.');elseif($da=="")query_redirect("CREATE SEQUENCE ".idf_escape($C),$_,'Sequence has been created.');elseif($da!=$C)query_redirect("ALTER SEQUENCE ".idf_escape($da)." RENAME TO ".idf_escape($C),$_,'Sequence has been altered.');else
redirect($_);}page_header($da!=""?'Alter sequence'.": ".h($da):'Create sequence',$l);if(!$K)$K["name"]=$da;echo'
<form action="" method="post">
<p><input name="name" value="',h($K["name"]),'" autocapitalize="off">
<input type="submit" value="Save">
';if($da!="")echo"<input type='submit' name='drop' value='".'Drop'."'>".confirm(sprintf('Drop %s?',$da))."\n";echo
input_token(),'</form>
';}elseif(isset($_GET["type"])){$ea=$_GET["type"];$K=$_POST;if($_POST&&!$l){$_=substr(ME,0,-1);if($_POST["drop"])query_redirect("DROP TYPE ".idf_escape($ea),$_,'Type has been dropped.');else
query_redirect("CREATE TYPE ".idf_escape(trim($K["name"]))." $K[as]",$_,'Type has been created.');}page_header($ea!=""?'Alter type'.": ".h($ea):'Create type',$l);if(!$K)$K["as"]="AS ";echo'
<form action="" method="post">
<p>
';if($ea!=""){$Si=driver()->types();$_c=type_values($Si[$ea]);if($_c)echo"<code class='jush-".JUSH."'>ENUM (".h($_c).")</code>\n<p>";echo"<input type='submit' name='drop' value='".'Drop'."'>".confirm(sprintf('Drop %s?',$ea))."\n";}else{echo'Name'.": <input name='name' value='".h($K['name'])."' autocapitalize='off'>\n",doc_link(array('pgsql'=>"datatype-enum.html",),"?");textarea("as",$K["as"]);echo"<p><input type='submit' value='".'Save'."'>\n";}echo
input_token(),'</form>
';}elseif(isset($_GET["check"])){$a=$_GET["check"];$C=$_GET["name"];$K=$_POST;if($K&&!$l){if(JUSH=="sqlite")$I=recreate_table($a,$a,array(),array(),array(),"",array(),"$C",($K["drop"]?"":$K["clause"]));else{$I=($C==""||queries("ALTER TABLE ".table($a)." DROP CONSTRAINT ".idf_escape($C)));if(!$K["drop"])$I=queries("ALTER TABLE ".table($a)." ADD".($K["name"]!=""?" CONSTRAINT ".idf_escape($K["name"]):"")." CHECK ($K[clause])");}queries_redirect(ME."table=".urlencode($a),($K["drop"]?'Check has been dropped.':($C!=""?'Check has been altered.':'Check has been created.')),$I);}page_header(($C!=""?'Alter check'.": ".h($C):'Create check'),$l,array("table"=>$a));if(!$K){$Za=driver()->checkConstraints($a);$K=array("name"=>$C,"clause"=>$Za[$C]);}echo'
<form action="" method="post">
<p>';if(JUSH!="sqlite")echo'Name'.': <input name="name" value="'.h($K["name"]).'" data-maxlength="64" autocapitalize="off"> ';echo
doc_link(array('sql'=>"create-table-check-constraints.html",'mariadb'=>"constraint/",'pgsql'=>"ddl-constraints.html#DDL-CONSTRAINTS-CHECK-CONSTRAINTS",'mssql'=>"relational-databases/tables/create-check-constraints",'sqlite'=>"lang_createtable.html#check_constraints",),"?"),'<p>';textarea("clause",$K["clause"]);echo'<p><input type="submit" value="Save">
';if($C!="")echo'<input type="submit" name="drop" value="Drop">',confirm(sprintf('Drop %s?',$C));echo
input_token(),'</form>
';}elseif(isset($_GET["trigger"])){$a=$_GET["trigger"];$C="$_GET[name]";$Oi=trigger_options();$K=(array)trigger($C,$a)+array("Trigger"=>$a."_bi");if($_POST){if(!$l&&in_array($_POST["Timing"],$Oi["Timing"])&&in_array($_POST["Event"],$Oi["Event"])&&in_array($_POST["Type"],$Oi["Type"])){$Cf=" ON ".table($a);$fc="DROP TRIGGER ".idf_escape($C).(JUSH=="pgsql"?$Cf:"");$A=ME."table=".urlencode($a);if($_POST["drop"])query_redirect($fc,$A,'Trigger has been dropped.');else{if($C!="")queries($fc);queries_redirect($A,($C!=""?'Trigger has been altered.':'Trigger has been created.'),queries(create_trigger($Cf,$_POST)));if($C!="")queries(create_trigger($Cf,$K+array("Type"=>reset($Oi["Type"]))));}}$K=$_POST;}page_header(($C!=""?'Alter trigger'.": ".h($C):'Create trigger'),$l,array("table"=>$a));echo'
<form action="" method="post" id="form">
<table class="layout">
<tr><th>Time<td>',html_select("Timing",$Oi["Timing"],$K["Timing"],"triggerChange(/^".preg_quote($a,"/")."_[ba][iud]$/, '".js_escape($a)."', this.form);"),'<tr><th>Event<td>',html_select("Event",$Oi["Event"],$K["Event"],"this.form['Timing'].onchange();"),(in_array("UPDATE OF",$Oi["Event"])?" <input name='Of' value='".h($K["Of"])."' class='hidden'>":""),'<tr><th>Type<td>',html_select("Type",$Oi["Type"],$K["Type"]),'</table>
<p>Name: <input name="Trigger" value="',h($K["Trigger"]),'" data-maxlength="64" autocapitalize="off">
',script("qs('#form')['Timing'].onchange();"),'<p>';textarea("Statement",$K["Statement"]);echo'<p>
<input type="submit" value="Save">
';if($C!="")echo'<input type="submit" name="drop" value="Drop">',confirm(sprintf('Drop %s?',$C));echo
input_token(),'</form>
';}elseif(isset($_GET["user"])){$fa=$_GET["user"];$Fg=array(""=>array("All privileges"=>""));foreach(get_rows("SHOW PRIVILEGES")as$K){foreach(explode(",",($K["Privilege"]=="Grant option"?"":$K["Context"]))as$yb)$Fg[$yb][$K["Privilege"]]=$K["Comment"];}$Fg["Server Admin"]+=$Fg["File access on server"];$Fg["Databases"]["Create routine"]=$Fg["Procedures"]["Create routine"];unset($Fg["Procedures"]["Create routine"]);$Fg["Columns"]=array();foreach(array("Select","Insert","Update","References")as$X)$Fg["Columns"][$X]=$Fg["Tables"][$X];unset($Fg["Server Admin"]["Usage"]);foreach($Fg["Tables"]as$x=>$X)unset($Fg["Databases"][$x]);$mf=array();if($_POST){foreach($_POST["objects"]as$x=>$X)$mf[$X]=(array)$mf[$X]+idx($_POST["grants"],$x,array());}$sd=array();$Af="";if(isset($_GET["host"])&&($I=connection()->query("SHOW GRANTS FOR ".q($fa)."@".q($_GET["host"])))){while($K=$I->fetch_row()){if(preg_match('~GRANT (.*) ON (.*) TO ~',$K[0],$B)&&preg_match_all('~ *([^(,]*[^ ,(])( *\([^)]+\))?~',$B[1],$Ne,PREG_SET_ORDER)){foreach($Ne
as$X){if($X[1]!="USAGE")$sd["$B[2]$X[2]"][$X[1]]=true;if(preg_match('~ WITH GRANT OPTION~',$K[0]))$sd["$B[2]$X[2]"]["GRANT OPTION"]=true;}}if(preg_match("~ IDENTIFIED BY PASSWORD '([^']+)~",$K[0],$B))$Af=$B[1];}}if($_POST&&!$l){$Bf=(isset($_GET["host"])?q($fa)."@".q($_GET["host"]):"''");if($_POST["drop"])query_redirect("DROP USER $Bf",ME."privileges=",'User has been dropped.');else{$of=q($_POST["user"])."@".q($_POST["host"]);$mg=$_POST["pass"];if($mg!=''&&!$_POST["hashed"]&&!min_version(8)){$mg=get_val("SELECT PASSWORD(".q($mg).")");$l=!$mg;}$Cb=false;if(!$l){if($Bf!=$of){$Cb=queries((min_version(5)?"CREATE USER":"GRANT USAGE ON *.* TO")." $of IDENTIFIED BY ".(min_version(8)?"":"PASSWORD ").q($mg));$l=!$Cb;}elseif($mg!=$Af)queries("SET PASSWORD FOR $of = ".q($mg));}if(!$l){$dh=array();foreach($mf
as$vf=>$rd){if(isset($_GET["grant"]))$rd=array_filter($rd);$rd=array_keys($rd);if(isset($_GET["grant"]))$dh=array_diff(array_keys(array_filter($mf[$vf],'strlen')),$rd);elseif($Bf==$of){$zf=array_keys((array)$sd[$vf]);$dh=array_diff($zf,$rd);$rd=array_diff($rd,$zf);unset($sd[$vf]);}if(preg_match('~^(.+)\s*(\(.*\))?$~U',$vf,$B)&&(!grant("REVOKE",$dh,$B[2]," ON $B[1] FROM $of")||!grant("GRANT",$rd,$B[2]," ON $B[1] TO $of"))){$l=true;break;}}}if(!$l&&isset($_GET["host"])){if($Bf!=$of)queries("DROP USER $Bf");elseif(!isset($_GET["grant"])){foreach($sd
as$vf=>$dh){if(preg_match('~^(.+)(\(.*\))?$~U',$vf,$B))grant("REVOKE",array_keys($dh),$B[2]," ON $B[1] FROM $of");}}}queries_redirect(ME."privileges=",(isset($_GET["host"])?'User has been altered.':'User has been created.'),!$l);if($Cb)connection()->query("DROP USER $of");}}page_header((isset($_GET["host"])?'Username'.": ".h("$fa@$_GET[host]"):'Create user'),$l,array("privileges"=>array('','Privileges')));$K=$_POST;if($K)$sd=$mf;else{$K=$_GET+array("host"=>get_val("SELECT SUBSTRING_INDEX(CURRENT_USER, '@', -1)"));$K["pass"]=$Af;if($Af!="")$K["hashed"]=true;$sd[(DB==""||$sd?"":idf_escape(addcslashes(DB,"%_\\"))).".*"]=array();}echo'<form action="" method="post">
<table class="layout">
<tr><th>Server<td><input name="host" data-maxlength="60" value="',h($K["host"]),'" autocapitalize="off">
<tr><th>Username<td><input name="user" data-maxlength="80" value="',h($K["user"]),'" autocapitalize="off">
<tr><th>Password<td><input name="pass" id="pass" value="',h($K["pass"]),'" autocomplete="new-password">
',($K["hashed"]?"":script("typePassword(qs('#pass'));")),(min_version(8)?"":checkbox("hashed",1,$K["hashed"],'Hashed',"typePassword(this.form['pass'], this.checked);")),'</table>

',"<table class='odds'>\n","<thead><tr><th colspan='2'>".'Privileges'.doc_link(array('sql'=>"grant.html#priv_level"));$s=0;foreach($sd
as$vf=>$rd){echo'<th>'.($vf!="*.*"?"<input name='objects[$s]' value='".h($vf)."' size='10' autocapitalize='off'>":input_hidden("objects[$s]","*.*")."*.*");$s++;}echo"</thead>\n";foreach(array(""=>"","Server Admin"=>'Server',"Databases"=>'Database',"Tables"=>'Table',"Columns"=>'Column',"Procedures"=>'Routine',)as$yb=>$Vb){foreach((array)$Fg[$yb]as$Eg=>$mb){echo"<tr><td".($Vb?">$Vb<td":" colspan='2'").' lang="en" title="'.h($mb).'">'.h($Eg);$s=0;foreach($sd
as$vf=>$rd){$C="'grants[$s][".h(strtoupper($Eg))."]'";$Y=$rd[strtoupper($Eg)];if($yb=="Server Admin"&&$vf!=(isset($sd["*.*"])?"*.*":".*"))echo"<td>";elseif(isset($_GET["grant"]))echo"<td><select name=$C><option><option value='1'".($Y?" selected":"").">".'Grant'."<option value='0'".($Y=="0"?" selected":"").">".'Revoke'."</select>";else
echo"<td align='center'><label class='block'>","<input type='checkbox' name=$C value='1'".($Y?" checked":"").($Eg=="All privileges"?" id='grants-$s-all'>":">".($Eg=="Grant option"?"":script("qsl('input').onclick = function () { if (this.checked) formUncheck('grants-$s-all'); };"))),"</label>";$s++;}}}echo"</table>\n",'<p>
<input type="submit" value="Save">
';if(isset($_GET["host"]))echo'<input type="submit" name="drop" value="Drop">',confirm(sprintf('Drop %s?',"$fa@$_GET[host]"));echo
input_token(),'</form>
';}elseif(isset($_GET["processlist"])){if(support("kill")){if($_POST&&!$l){$ue=0;foreach((array)$_POST["kill"]as$X){if(kill_process($X))$ue++;}queries_redirect(ME."processlist=",lang_format(array('%d process has been killed.','%d processes have been killed.'),$ue),$ue||!$_POST["kill"]);}}page_header('Process list',$l);echo'
<form action="" method="post">
<div class="scrollable">
<table class="nowrap checkable odds">
',script("mixin(qsl('table'), {onclick: tableClick, ondblclick: partialArg(tableClick, true)});");$s=-1;foreach(process_list()as$s=>$K){if(!$s){echo"<thead><tr lang='en'>".(support("kill")?"<th>":"");foreach($K
as$x=>$X)echo"<th>$x".doc_link(array('sql'=>"show-processlist.html#processlist_".strtolower($x),'pgsql'=>"monitoring-stats.html#PG-STAT-ACTIVITY-VIEW",'oracle'=>"REFRN30223",));echo"</thead>\n";}echo"<tr>".(support("kill")?"<td>".checkbox("kill[]",$K[JUSH=="sql"?"Id":"pid"],0):"");foreach($K
as$x=>$X)echo"<td>".((JUSH=="sql"&&$x=="Info"&&preg_match("~Query|Killed~",$K["Command"])&&$X!="")||(JUSH=="pgsql"&&$x=="current_query"&&$X!="<IDLE>")||(JUSH=="oracle"&&$x=="sql_text"&&$X!="")?"<code class='jush-".JUSH."'>".shorten_utf8($X,100,"</code>").' <a href="'.h(ME.($K["db"]!=""?"db=".urlencode($K["db"])."&":"")."sql=".urlencode($X)).'">'.'Clone'.'</a>':h($X));echo"\n";}echo'</table>
</div>
<p>
';if(support("kill"))echo($s+1)."/".sprintf('%d in total',max_connections()),"<p><input type='submit' value='".'Kill'."'>\n";echo
input_token(),'</form>
',script("tableCheck();");}elseif(isset($_GET["select"])){$a=$_GET["select"];$S=table_status1($a);$w=indexes($a);$n=fields($a);$id=column_foreign_keys($a);$xf=$S["Oid"];$na=get_settings("adminer_import");$eh=array();$e=array();$rh=array();$Of=array();$ui="";foreach($n
as$x=>$m){$C=adminer()->fieldName($m);$kf=html_entity_decode(strip_tags($C),ENT_QUOTES);if(isset($m["privileges"]["select"])&&$C!=""){$e[$x]=$kf;if(is_shortable($m))$ui=adminer()->selectLengthProcess();}if(isset($m["privileges"]["where"])&&$C!="")$rh[$x]=$kf;if(isset($m["privileges"]["order"])&&$C!="")$Of[$x]=$kf;$eh+=$m["privileges"];}list($M,$td)=adminer()->selectColumnsProcess($e,$w);$M=array_unique($M);$td=array_unique($td);$ke=count($td)<count($M);$Z=adminer()->selectSearchProcess($n,$w);$Nf=adminer()->selectOrderProcess($n,$w);$z=adminer()->selectLimitProcess();if($_GET["val"]&&is_ajax()){header("Content-Type: text/plain; charset=utf-8");foreach($_GET["val"]as$Xi=>$K){$wa=convert_field($n[key($K)]);$M=array($wa?:idf_escape(key($K)));$Z[]=where_check($Xi,$n);$J=driver()->select($a,$M,$Z,$M);if($J)echo
first($J->fetch_row());}exit;}$G=$Zi=array();foreach($w
as$v){if($v["type"]=="PRIMARY"){$G=array_flip($v["columns"]);$Zi=($M?$G:array());foreach($Zi
as$x=>$X){if(in_array(idf_escape($x),$M))unset($Zi[$x]);}break;}}if($xf&&!$G){$G=$Zi=array($xf=>0);$w[]=array("type"=>"PRIMARY","columns"=>array($xf));}if($_POST&&!$l){$xj=$Z;if(!$_POST["all"]&&is_array($_POST["check"])){$Za=array();foreach($_POST["check"]as$Va)$Za[]=where_check($Va,$n);$xj[]="((".implode(") OR (",$Za)."))";}$xj=($xj?"\nWHERE ".implode(" AND ",$xj):"");if($_POST["export"]){save_settings(array("output"=>$_POST["output"],"format"=>$_POST["format"]),"adminer_import");dump_headers($a);adminer()->dumpTable($a,"");$md=($M?implode(", ",$M):"*").convert_fields($e,$n,$M)."\nFROM ".table($a);$vd=($td&&$ke?"\nGROUP BY ".implode(", ",$td):"").($Nf?"\nORDER BY ".implode(", ",$Nf):"");$H="SELECT $md$xj$vd";if(is_array($_POST["check"])&&!$G){$Vi=array();foreach($_POST["check"]as$X)$Vi[]="(SELECT".limit($md,"\nWHERE ".($Z?implode(" AND ",$Z)." AND ":"").where_check($X,$n).$vd,1).")";$H=implode(" UNION ALL ",$Vi);}adminer()->dumpData($a,"table",$H);adminer()->dumpFooter();exit;}if(!adminer()->selectEmailProcess($Z,$id)){if($_POST["save"]||$_POST["delete"]){$I=true;$oa=0;$O=array();if(!$_POST["delete"]){foreach($_POST["fields"]as$C=>$X){$X=process_input($n[$C]);if($X!==null&&($_POST["clone"]||$X!==false))$O[idf_escape($C)]=($X!==false?$X:idf_escape($C));}}if($_POST["delete"]||$O){$H=($_POST["clone"]?"INTO ".table($a)." (".implode(", ",array_keys($O)).")\nSELECT ".implode(", ",$O)."\nFROM ".table($a):"");if($_POST["all"]||($G&&is_array($_POST["check"]))||$ke){$I=($_POST["delete"]?driver()->delete($a,$xj):($_POST["clone"]?queries("INSERT $H$xj".driver()->insertReturning($a)):driver()->update($a,$O,$xj)));$oa=connection()->affected_rows;if(is_object($I))$oa+=$I->num_rows;}else{foreach((array)$_POST["check"]as$X){$wj="\nWHERE ".($Z?implode(" AND ",$Z)." AND ":"").where_check($X,$n);$I=($_POST["delete"]?driver()->delete($a,$wj,1):($_POST["clone"]?queries("INSERT".limit1($a,$H,$wj)):driver()->update($a,$O,$wj,1)));if(!$I)break;$oa+=connection()->affected_rows;}}}$Xe=lang_format(array('%d item has been affected.','%d items have been affected.'),$oa);if($_POST["clone"]&&$I&&$oa==1){$ze=last_id($I);if($ze)$Xe=sprintf('Item%s has been inserted.'," $ze");}queries_redirect(remove_from_uri($_POST["all"]&&$_POST["delete"]?"page":""),$Xe,$I);if(!$_POST["delete"]){$yg=(array)$_POST["fields"];edit_form($a,array_intersect_key($n,$yg),$yg,!$_POST["clone"],$l);page_footer();exit;}}elseif(!$_POST["import"]){if(!$_POST["val"])$l='Ctrl+click on a value to modify it.';else{$I=true;$oa=0;foreach($_POST["val"]as$Xi=>$K){$O=array();foreach($K
as$x=>$X){$x=bracket_escape($x,true);$O[idf_escape($x)]=(preg_match('~char|text~',$n[$x]["type"])||$X!=""?adminer()->processInput($n[$x],$X):"NULL");}$I=driver()->update($a,$O," WHERE ".($Z?implode(" AND ",$Z)." AND ":"").where_check($Xi,$n),($ke||$G?0:1)," ");if(!$I)break;$oa+=connection()->affected_rows;}queries_redirect(remove_from_uri(),lang_format(array('%d item has been affected.','%d items have been affected.'),$oa),$I);}}elseif(!is_string($Wc=get_file("csv_file",true)))$l=upload_error($Wc);elseif(!preg_match('~~u',$Wc))$l='File must be in UTF-8 encoding.';else{save_settings(array("output"=>$na["output"],"format"=>$_POST["separator"]),"adminer_import");$I=true;$ib=array_keys($n);preg_match_all('~(?>"[^"]*"|[^"\r\n]+)+~',$Wc,$Ne);$oa=count($Ne[0]);driver()->begin();$xh=($_POST["separator"]=="csv"?",":($_POST["separator"]=="tsv"?"\t":";"));$L=array();foreach($Ne[0]as$x=>$X){preg_match_all("~((?>\"[^\"]*\")+|[^$xh]*)$xh~",$X.$xh,$Oe);if(!$x&&!array_diff($Oe[1],$ib)){$ib=$Oe[1];$oa--;}else{$O=array();foreach($Oe[1]as$s=>$fb)$O[idf_escape($ib[$s])]=($fb==""&&$n[$ib[$s]]["null"]?"NULL":q(preg_match('~^".*"$~s',$fb)?str_replace('""','"',substr($fb,1,-1)):$fb));$L[]=$O;}}$I=(!$L||driver()->insertUpdate($a,$L,$G));if($I)driver()->commit();queries_redirect(remove_from_uri("page"),lang_format(array('%d row has been imported.','%d rows have been imported.'),$oa),$I);driver()->rollback();}}}$fi=adminer()->tableName($S);if(is_ajax()){page_headers();ob_start();}else
page_header('Select'.": $fi",$l);$O=null;if(isset($eh["insert"])||!support("table")){$fg=array();foreach((array)$_GET["where"]as$X){if(isset($id[$X["col"]])&&count($id[$X["col"]])==1&&($X["op"]=="="||(!$X["op"]&&(is_array($X["val"])||!preg_match('~[_%]~',$X["val"])))))$fg["set"."[".bracket_escape($X["col"])."]"]=$X["val"];}$O=$fg?"&".http_build_query($fg):"";}adminer()->selectLinks($S,$O);if(!$e&&support("table"))echo"<p class='error'>".'Unable to select the table'.($n?".":": ".error())."\n";else{echo"<form action='' id='form'>\n","<div style='display: none;'>";hidden_fields_get();echo(DB!=""?input_hidden("db",DB).(isset($_GET["ns"])?input_hidden("ns",$_GET["ns"]):""):""),input_hidden("select",$a),"</div>\n";adminer()->selectColumnsPrint($M,$e);adminer()->selectSearchPrint($Z,$rh,$w);adminer()->selectOrderPrint($Nf,$Of,$w);adminer()->selectLimitPrint($z);adminer()->selectLengthPrint($ui);adminer()->selectActionPrint($w);echo"</form>\n";$E=$_GET["page"];$ld=null;if($E=="last"){$ld=get_val(count_rows($a,$Z,$ke,$td));$E=floor(max(0,intval($ld)-1)/$z);}$sh=$M;$ud=$td;if(!$sh){$sh[]="*";$zb=convert_fields($e,$n,$M);if($zb)$sh[]=substr($zb,2);}foreach($M
as$x=>$X){$m=$n[idf_unescape($X)];if($m&&($wa=convert_field($m)))$sh[$x]="$wa AS $X";}if(!$ke&&$Zi){foreach($Zi
as$x=>$X){$sh[]=idf_escape($x);if($ud)$ud[]=idf_escape($x);}}$I=driver()->select($a,$sh,$Z,$ud,$Nf,$z,$E,true);if(!$I)echo"<p class='error'>".error()."\n";else{if(JUSH=="mssql"&&$E)$I->seek($z*$E);$tc=array();echo"<form action='' method='post' enctype='multipart/form-data'>\n";$L=array();while($K=$I->fetch_assoc()){if($E&&JUSH=="oracle")unset($K["RNUM"]);$L[]=$K;}if($_GET["page"]!="last"&&$z&&$td&&$ke&&JUSH=="sql")$ld=get_val(" SELECT FOUND_ROWS()");if(!$L)echo"<p class='message'>".'No rows.'."\n";else{$Ea=adminer()->backwardKeys($a,$fi);echo"<div class='scrollable'>","<table id='table' class='nowrap checkable odds'>",script("mixin(qs('#table'), {onclick: tableClick, ondblclick: partialArg(tableClick, true), onkeydown: editingKeydown});"),"<thead><tr>".(!$td&&$M?"":"<td><input type='checkbox' id='all-page' class='jsonly'>".script("qs('#all-page').onclick = partial(formCheck, /check/);","")." <a href='".h($_GET["modify"]?remove_from_uri("modify"):$_SERVER["REQUEST_URI"]."&modify=1")."'>".'Modify'."</a>");$lf=array();$od=array();reset($M);$Og=1;foreach($L[0]as$x=>$X){if(!isset($Zi[$x])){$X=idx($_GET["columns"],key($M))?:array();$m=$n[$M?($X?$X["col"]:current($M)):$x];$C=($m?adminer()->fieldName($m,$Og):($X["fun"]?"*":h($x)));if($C!=""){$Og++;$lf[$x]=$C;$d=idf_escape($x);$Kd=remove_from_uri('(order|desc)[^=]*|page').'&order%5B0%5D='.urlencode($x);$Vb="&desc%5B0%5D=1";echo"<th id='th[".h(bracket_escape($x))."]'>".script("mixin(qsl('th'), {onmouseover: partial(columnMouse), onmouseout: partial(columnMouse, ' hidden')});","");$nd=apply_sql_function($X["fun"],$C);$Jh=isset($m["privileges"]["order"])||$nd;echo($Jh?'<a href="'.h($Kd.($Nf[0]==$d||$Nf[0]==$x||(!$Nf&&$ke&&$td[0]==$d)?$Vb:'')).'">'."$nd</a>":$nd),"<span class='column hidden'>";if($Jh)echo"<a href='".h($Kd.$Vb)."' title='".'descending'."' class='text'> ↓</a>";if(!$X["fun"]&&isset($m["privileges"]["where"]))echo'<a href="#fieldset-search" title="'.'Search'.'" class="text jsonly"> =</a>',script("qsl('a').onclick = partial(selectSearch, '".js_escape($x)."');");echo"</span>";}$od[$x]=$X["fun"];next($M);}}$De=array();if($_GET["modify"]){foreach($L
as$K){foreach($K
as$x=>$X)$De[$x]=max($De[$x],min(40,strlen(utf8_decode($X))));}}echo($Ea?"<th>".'Relations':"")."</thead>\n";if(is_ajax())ob_end_clean();foreach(adminer()->rowDescriptions($L,$id)as$jf=>$K){$Wi=unique_array($L[$jf],$w);if(!$Wi){$Wi=array();foreach($L[$jf]as$x=>$X){if(!preg_match('~^(COUNT\((\*|(DISTINCT )?`(?:[^`]|``)+`)\)|(AVG|GROUP_CONCAT|MAX|MIN|SUM)\(`(?:[^`]|``)+`\))$~',$x))$Wi[$x]=$X;}}$Xi="";foreach($Wi
as$x=>$X){$m=(array)$n[$x];if((JUSH=="sql"||JUSH=="pgsql")&&preg_match('~char|text|enum|set~',$m["type"])&&strlen($X)>64){$x=(strpos($x,'(')?$x:idf_escape($x));$x="MD5(".(JUSH!='sql'||preg_match("~^utf8~",$m["collation"])?$x:"CONVERT($x USING ".charset(connection()).")").")";$X=md5($X);}$Xi
.="&".($X!==null?urlencode("where[".bracket_escape($x)."]")."=".urlencode($X===false?"f":$X):"null%5B%5D=".urlencode($x));}echo"<tr>".(!$td&&$M?"":"<td>".checkbox("check[]",substr($Xi,1),in_array(substr($Xi,1),(array)$_POST["check"])).($ke||information_schema(DB)?"":" <a href='".h(ME."edit=".urlencode($a).$Xi)."' class='edit'>".'edit'."</a>"));foreach($K
as$x=>$X){if(isset($lf[$x])){$m=(array)$n[$x];$X=driver()->value($X,$m);if($X!=""&&(!isset($tc[$x])||$tc[$x]!=""))$tc[$x]=(is_mail($X)?$lf[$x]:"");$_="";if(preg_match('~blob|bytea|raw|file~',$m["type"])&&$X!="")$_=ME.'download='.urlencode($a).'&field='.urlencode($x).$Xi;if(!$_&&$X!==null){foreach((array)$id[$x]as$p){if(count($id[$x])==1||end($p["source"])==$x){$_="";foreach($p["source"]as$s=>$Kh)$_
.=where_link($s,$p["target"][$s],$L[$jf][$Kh]);$_=($p["db"]!=""?preg_replace('~([?&]db=)[^&]+~','\1'.urlencode($p["db"]),ME):ME).'select='.urlencode($p["table"]).$_;if($p["ns"])$_=preg_replace('~([?&]ns=)[^&]+~','\1'.urlencode($p["ns"]),$_);if(count($p["source"])==1)break;}}}if($x=="COUNT(*)"){$_=ME."select=".urlencode($a);$s=0;foreach((array)$_GET["where"]as$W){if(!array_key_exists($W["col"],$Wi))$_
.=where_link($s++,$W["col"],$W["val"],$W["op"]);}foreach($Wi
as$qe=>$W)$_
.=where_link($s++,$qe,$W);}$Ld=select_value($X,$_,$m,$ui);$t=h("val[$Xi][".bracket_escape($x)."]");$zg=idx(idx($_POST["val"],$Xi),bracket_escape($x));$oc=!is_array($K[$x])&&is_utf8($Ld)&&$L[$jf][$x]==$K[$x]&&!$od[$x]&&!$m["generated"];$si=preg_match('~text|json|lob~',$m["type"]);echo"<td id='$t'".(preg_match(number_type(),$m["type"])&&($X===null||is_numeric(strip_tags($Ld)))?" class='number'":"");if(($_GET["modify"]&&$oc&&$X!==null)||$zg!==null){$yd=h($zg!==null?$zg:$K[$x]);echo">".($si?"<textarea name='$t' cols='30' rows='".(substr_count($K[$x],"\n")+1)."'>$yd</textarea>":"<input name='$t' value='$yd' size='$De[$x]'>");}else{$Ie=strpos($Ld,"<i>…</i>");echo" data-text='".($Ie?2:($si?1:0))."'".($oc?"":" data-warning='".h('Use edit link to modify this value.')."'").">$Ld";}}}if($Ea)echo"<td>";adminer()->backwardKeysPrint($Ea,$L[$jf]);echo"</tr>\n";}if(is_ajax())exit;echo"</table>\n","</div>\n";}if(!is_ajax()){if($L||$E){$Gc=true;if($_GET["page"]!="last"){if(!$z||(count($L)<$z&&($L||!$E)))$ld=($E?$E*$z:0)+count($L);elseif(JUSH!="sql"||!$ke){$ld=($ke?false:found_rows($S,$Z));if(intval($ld)<max(1e4,2*($E+1)*$z))$ld=first(slow_query(count_rows($a,$Z,$ke,$td)));else$Gc=false;}}$dg=($z&&($ld===false||$ld>$z||$E));if($dg)echo(($ld===false?count($L)+1:$ld-$E*$z)>$z?'<p><a href="'.h(remove_from_uri("page")."&page=".($E+1)).'" class="loadmore">'.'Load more data'.'</a>'.script("qsl('a').onclick = partial(selectLoadMore, $z, '".'Loading'."…');",""):''),"\n";echo"<div class='footer'><div>\n";if($dg){$Qe=($ld===false?$E+(count($L)>=$z?2:1):floor(($ld-1)/$z));echo"<fieldset>";if(JUSH!="simpledb"){echo"<legend><a href='".h(remove_from_uri("page"))."'>".'Page'."</a></legend>",script("qsl('a').onclick = function () { pageClick(this.href, +prompt('".'Page'."', '".($E+1)."')); return false; };"),pagination(0,$E).($E>5?" …":"");for($s=max(1,$E-4);$s<min($Qe,$E+5);$s++)echo
pagination($s,$E);if($Qe>0)echo($E+5<$Qe?" …":""),($Gc&&$ld!==false?pagination($Qe,$E):" <a href='".h(remove_from_uri("page")."&page=last")."' title='~$Qe'>".'last'."</a>");}else
echo"<legend>".'Page'."</legend>",pagination(0,$E).($E>1?" …":""),($E?pagination($E,$E):""),($Qe>$E?pagination($E+1,$E).($Qe>$E+1?" …":""):"");echo"</fieldset>\n";}echo"<fieldset>","<legend>".'Whole result'."</legend>";$cc=($Gc?"":"~ ").$ld;$Gf="const checked = formChecked(this, /check/); selectCount('selected', this.checked ? '$cc' : checked); selectCount('selected2', this.checked || !checked ? '$cc' : checked);";echo
checkbox("all",1,0,($ld!==false?($Gc?"":"~ ").lang_format(array('%d row','%d rows'),$ld):""),$Gf)."\n","</fieldset>\n";if(adminer()->selectCommandPrint())echo'<fieldset',($_GET["modify"]?'':' class="jsonly"'),'><legend>Modify</legend><div>
<input type="submit" value="Save"',($_GET["modify"]?'':' title="'.'Ctrl+click on a value to modify it.'.'"'),'>
</div></fieldset>
<fieldset><legend>Selected <span id="selected"></span></legend><div>
<input type="submit" name="edit" value="Edit">
<input type="submit" name="clone" value="Clone">
<input type="submit" name="delete" value="Delete">',confirm(),'</div></fieldset>
';$jd=adminer()->dumpFormat();foreach((array)$_GET["columns"]as$d){if($d["fun"]){unset($jd['sql']);break;}}if($jd){print_fieldset("export",'Export'." <span id='selected2'></span>");$ag=adminer()->dumpOutput();echo($ag?html_select("output",$ag,$na["output"])." ":""),html_select("format",$jd,$na["format"])," <input type='submit' name='export' value='".'Export'."'>\n","</div></fieldset>\n";}adminer()->selectEmailPrint(array_filter($tc,'strlen'),$e);echo"</div></div>\n";}if(adminer()->selectImportPrint())echo"<p>","<a href='#import'>".'Import'."</a>",script("qsl('a').onclick = partial(toggle, 'import');",""),"<span id='import'".($_POST["import"]?"":" class='hidden'").">: ","<input type='file' name='csv_file'> ",html_select("separator",array("csv"=>"CSV,","csv;"=>"CSV;","tsv"=>"TSV"),$na["format"])," <input type='submit' name='import' value='".'Import'."'>","</span>";echo
input_token(),"</form>\n",(!$td&&$M?"":script("tableCheck();"));}}}if(is_ajax()){ob_end_clean();exit;}}elseif(isset($_GET["variables"])){$P=isset($_GET["status"]);page_header($P?'Status':'Variables');$nj=($P?show_status():show_variables());if(!$nj)echo"<p class='message'>".'No rows.'."\n";else{echo"<table>\n";foreach($nj
as$K){echo"<tr>";$x=array_shift($K);echo"<th><code class='jush-".JUSH.($P?"status":"set")."'>".h($x)."</code>";foreach($K
as$X)echo"<td>".nl_br(h($X));}echo"</table>\n";}}elseif(isset($_GET["script"])){header("Content-Type: text/javascript; charset=utf-8");if($_GET["script"]=="db"){$bi=array("Data_length"=>0,"Index_length"=>0,"Data_free"=>0);foreach(table_status()as$C=>$S){json_row("Comment-$C",h($S["Comment"]));if(!is_view($S)){foreach(array("Engine","Collation")as$x)json_row("$x-$C",h($S[$x]));foreach($bi+array("Auto_increment"=>0,"Rows"=>0)as$x=>$X){if($S[$x]!=""){$X=format_number($S[$x]);if($X>=0)json_row("$x-$C",($x=="Rows"&&$X&&$S["Engine"]==(JUSH=="pgsql"?"table":"InnoDB")?"~ $X":$X));if(isset($bi[$x]))$bi[$x]+=($S["Engine"]!="InnoDB"||$x!="Data_free"?$S[$x]:0);}elseif(array_key_exists($x,$S))json_row("$x-$C","?");}}}foreach($bi
as$x=>$X)json_row("sum-$x",format_number($X));json_row("");}elseif($_GET["script"]=="kill")connection()->query("KILL ".number($_POST["kill"]));else{foreach(count_tables(adminer()->databases())as$j=>$X){json_row("tables-$j",$X);json_row("size-$j",db_size($j));}json_row("");}exit;}else{$mi=array_merge((array)$_POST["tables"],(array)$_POST["views"]);if($mi&&!$l&&!$_POST["search"]){$I=true;$Xe="";if(JUSH=="sql"&&$_POST["tables"]&&count($_POST["tables"])>1&&($_POST["drop"]||$_POST["truncate"]||$_POST["copy"]))queries("SET foreign_key_checks = 0");if($_POST["truncate"]){if($_POST["tables"])$I=truncate_tables($_POST["tables"]);$Xe='Tables have been truncated.';}elseif($_POST["move"]){$I=move_tables((array)$_POST["tables"],(array)$_POST["views"],$_POST["target"]);$Xe='Tables have been moved.';}elseif($_POST["copy"]){$I=copy_tables((array)$_POST["tables"],(array)$_POST["views"],$_POST["target"]);$Xe='Tables have been copied.';}elseif($_POST["drop"]){if($_POST["views"])$I=drop_views($_POST["views"]);if($I&&$_POST["tables"])$I=drop_tables($_POST["tables"]);$Xe='Tables have been dropped.';}elseif(JUSH=="sqlite"&&$_POST["check"]){foreach((array)$_POST["tables"]as$R){foreach(get_rows("PRAGMA integrity_check(".q($R).")")as$K)$Xe
.="<b>".h($R)."</b>: ".h($K["integrity_check"])."<br>";}}elseif(JUSH!="sql"){$I=(JUSH=="sqlite"?queries("VACUUM"):apply_queries("VACUUM".($_POST["optimize"]?"":" ANALYZE"),$_POST["tables"]));$Xe='Tables have been optimized.';}elseif(!$_POST["tables"])$Xe='No tables.';elseif($I=queries(($_POST["optimize"]?"OPTIMIZE":($_POST["check"]?"CHECK":($_POST["repair"]?"REPAIR":"ANALYZE")))." TABLE ".implode(", ",array_map('Adminer\idf_escape',$_POST["tables"])))){while($K=$I->fetch_assoc())$Xe
.="<b>".h($K["Table"])."</b>: ".h($K["Msg_text"])."<br>";}queries_redirect(substr(ME,0,-1),$Xe,$I);}page_header(($_GET["ns"]==""?'Database'.": ".h(DB):'Schema'.": ".h($_GET["ns"])),$l,true);if(adminer()->homepage()){if($_GET["ns"]!==""){echo"<h3 id='tables-views'>".'Tables and views'."</h3>\n";$li=tables_list();if(!$li)echo"<p class='message'>".'No tables.'."\n";else{echo"<form action='' method='post'>\n";if(support("table")){echo"<fieldset><legend>".'Search data in tables'." <span id='selected2'></span></legend><div>","<input type='search' name='query' value='".h($_POST["query"])."'>",script("qsl('input').onkeydown = partialArg(bodyKeydown, 'search');","")," <input type='submit' name='search' value='".'Search'."'>\n","</div></fieldset>\n";if($_POST["search"]&&$_POST["query"]!=""){$_GET["where"][0]["op"]=driver()->convertOperator("LIKE %%");search_tables();}}echo"<div class='scrollable'>\n","<table class='nowrap checkable odds'>\n",script("mixin(qsl('table'), {onclick: tableClick, ondblclick: partialArg(tableClick, true)});"),'<thead><tr class="wrap">','<td><input id="check-all" type="checkbox" class="jsonly">'.script("qs('#check-all').onclick = partial(formCheck, /^(tables|views)\[/);",""),'<th>'.'Table','<td>'.'Engine'.doc_link(array('sql'=>'storage-engines.html')),'<td>'.'Collation'.doc_link(array('sql'=>'charset-charsets.html','mariadb'=>'supported-character-sets-and-collations/')),'<td>'.'Data Length'.doc_link(array('sql'=>'show-table-status.html','pgsql'=>'functions-admin.html#FUNCTIONS-ADMIN-DBOBJECT','oracle'=>'REFRN20286')),'<td>'.'Index Length'.doc_link(array('sql'=>'show-table-status.html','pgsql'=>'functions-admin.html#FUNCTIONS-ADMIN-DBOBJECT')),'<td>'.'Data Free'.doc_link(array('sql'=>'show-table-status.html')),'<td>'.'Auto Increment'.doc_link(array('sql'=>'example-auto-increment.html','mariadb'=>'auto_increment/')),'<td>'.'Rows'.doc_link(array('sql'=>'show-table-status.html','pgsql'=>'catalog-pg-class.html#CATALOG-PG-CLASS','oracle'=>'REFRN20286')),(support("comment")?'<td>'.'Comment'.doc_link(array('sql'=>'show-table-status.html','pgsql'=>'functions-info.html#FUNCTIONS-INFO-COMMENT-TABLE')):''),"</thead>\n";$T=0;foreach($li
as$C=>$U){$qj=($U!==null&&!preg_match('~table|sequence~i',$U));$t=h("Table-".$C);echo'<tr><td>'.checkbox(($qj?"views[]":"tables[]"),$C,in_array("$C",$mi,true),"","","",$t),'<th>'.(support("table")||support("indexes")?"<a href='".h(ME)."table=".urlencode($C)."' title='".'Show structure'."' id='$t'>".h($C).'</a>':h($C));if($qj)echo'<td colspan="6"><a href="'.h(ME)."view=".urlencode($C).'" title="'.'Alter view'.'">'.(preg_match('~materialized~i',$U)?'Materialized view':'View').'</a>','<td align="right"><a href="'.h(ME)."select=".urlencode($C).'" title="'.'Select data'.'">?</a>';else{foreach(array("Engine"=>array(),"Collation"=>array(),"Data_length"=>array("create",'Alter table'),"Index_length"=>array("indexes",'Alter indexes'),"Data_free"=>array("edit",'New item'),"Auto_increment"=>array("auto_increment=1&create",'Alter table'),"Rows"=>array("select",'Select data'),)as$x=>$_){$t=" id='$x-".h($C)."'";echo($_?"<td align='right'>".(support("table")||$x=="Rows"||(support("indexes")&&$x!="Data_length")?"<a href='".h(ME."$_[0]=").urlencode($C)."'$t title='$_[1]'>?</a>":"<span$t>?</span>"):"<td id='$x-".h($C)."'>");}$T++;}echo(support("comment")?"<td id='Comment-".h($C)."'>":""),"\n";}echo"<tr><td><th>".sprintf('%d in total',count($li)),"<td>".h(JUSH=="sql"?get_val("SELECT @@default_storage_engine"):""),"<td>".h(db_collation(DB,collations()));foreach(array("Data_length","Index_length","Data_free")as$x)echo"<td align='right' id='sum-$x'>";echo"\n","</table>\n","</div>\n";if(!information_schema(DB)){echo"<div class='footer'><div>\n";$kj="<input type='submit' value='".'Vacuum'."'> ".on_help("'VACUUM'");$Jf="<input type='submit' name='optimize' value='".'Optimize'."'> ".on_help(JUSH=="sql"?"'OPTIMIZE TABLE'":"'VACUUM OPTIMIZE'");echo"<fieldset><legend>".'Selected'." <span id='selected'></span></legend><div>".(JUSH=="sqlite"?$kj."<input type='submit' name='check' value='".'Check'."'> ".on_help("'PRAGMA integrity_check'"):(JUSH=="pgsql"?$kj.$Jf:(JUSH=="sql"?"<input type='submit' value='".'Analyze'."'> ".on_help("'ANALYZE TABLE'").$Jf."<input type='submit' name='check' value='".'Check'."'> ".on_help("'CHECK TABLE'")."<input type='submit' name='repair' value='".'Repair'."'> ".on_help("'REPAIR TABLE'"):"")))."<input type='submit' name='truncate' value='".'Truncate'."'> ".on_help(JUSH=="sqlite"?"'DELETE'":"'TRUNCATE".(JUSH=="pgsql"?"'":" TABLE'")).confirm()."<input type='submit' name='drop' value='".'Drop'."'>".on_help("'DROP TABLE'").confirm()."\n";$i=(support("scheme")?adminer()->schemas():adminer()->databases());if(count($i)!=1&&JUSH!="sqlite"){$j=(isset($_POST["target"])?$_POST["target"]:(support("scheme")?$_GET["ns"]:DB));echo"<p><label>".'Move to other database'.": ",($i?html_select("target",$i,$j):'<input name="target" value="'.h($j).'" autocapitalize="off">'),"</label> <input type='submit' name='move' value='".'Move'."'>",(support("copy")?" <input type='submit' name='copy' value='".'Copy'."'> ".checkbox("overwrite",1,$_POST["overwrite"],'overwrite'):""),"\n";}echo"<input type='hidden' name='all' value=''>",script("qsl('input').onclick = function () { selectCount('selected', formChecked(this, /^(tables|views)\[/));".(support("table")?" selectCount('selected2', formChecked(this, /^tables\[/) || $T);":"")." }"),input_token(),"</div></fieldset>\n","</div></div>\n";}echo"</form>\n",script("tableCheck();");}echo"<p class='links'><a href='".h(ME)."create='>".'Create table'."</a>\n",(support("view")?"<a href='".h(ME)."view='>".'Create view'."</a>\n":"");if(support("routine")){echo"<h3 id='routines'>".'Routines'."</h3>\n";$ih=routines();if($ih){echo"<table class='odds'>\n",'<thead><tr><th>'.'Name'.'<td>'.'Type'.'<td>'.'Return type'."<td></thead>\n";foreach($ih
as$K){$C=($K["SPECIFIC_NAME"]==$K["ROUTINE_NAME"]?"":"&name=".urlencode($K["ROUTINE_NAME"]));echo'<tr>','<th><a href="'.h(ME.($K["ROUTINE_TYPE"]!="PROCEDURE"?'callf=':'call=').urlencode($K["SPECIFIC_NAME"]).$C).'">'.h($K["ROUTINE_NAME"]).'</a>','<td>'.h($K["ROUTINE_TYPE"]),'<td>'.h($K["DTD_IDENTIFIER"]),'<td><a href="'.h(ME.($K["ROUTINE_TYPE"]!="PROCEDURE"?'function=':'procedure=').urlencode($K["SPECIFIC_NAME"]).$C).'">'.'Alter'."</a>";}echo"</table>\n";}echo'<p class="links">'.(support("procedure")?'<a href="'.h(ME).'procedure=">'.'Create procedure'.'</a>':'').'<a href="'.h(ME).'function=">'.'Create function'."</a>\n";}if(support("sequence")){echo"<h3 id='sequences'>".'Sequences'."</h3>\n";$_h=get_vals("SELECT sequence_name FROM information_schema.sequences WHERE sequence_schema = current_schema() ORDER BY sequence_name");if($_h){echo"<table class='odds'>\n","<thead><tr><th>".'Name'."</thead>\n";foreach($_h
as$X)echo"<tr><th><a href='".h(ME)."sequence=".urlencode($X)."'>".h($X)."</a>\n";echo"</table>\n";}echo"<p class='links'><a href='".h(ME)."sequence='>".'Create sequence'."</a>\n";}if(support("type")){echo"<h3 id='user-types'>".'User types'."</h3>\n";$ij=types();if($ij){echo"<table class='odds'>\n","<thead><tr><th>".'Name'."</thead>\n";foreach($ij
as$X)echo"<tr><th><a href='".h(ME)."type=".urlencode($X)."'>".h($X)."</a>\n";echo"</table>\n";}echo"<p class='links'><a href='".h(ME)."type='>".'Create type'."</a>\n";}if(support("event")){echo"<h3 id='events'>".'Events'."</h3>\n";$L=get_rows("SHOW EVENTS");if($L){echo"<table>\n","<thead><tr><th>".'Name'."<td>".'Schedule'."<td>".'Start'."<td>".'End'."<td></thead>\n";foreach($L
as$K)echo"<tr>","<th>".h($K["Name"]),"<td>".($K["Execute at"]?'At given time'."<td>".$K["Execute at"]:'Every'." ".$K["Interval value"]." ".$K["Interval field"]."<td>$K[Starts]"),"<td>$K[Ends]",'<td><a href="'.h(ME).'event='.urlencode($K["Name"]).'">'.'Alter'.'</a>';echo"</table>\n";$Ec=get_val("SELECT @@event_scheduler");if($Ec&&$Ec!="ON")echo"<p class='error'><code class='jush-sqlset'>event_scheduler</code>: ".h($Ec)."\n";}echo'<p class="links"><a href="'.h(ME).'event=">'.'Create event'."</a>\n";}if($li)echo
script("ajaxSetHtml('".js_escape(ME)."script=db');");}}}page_footer();
