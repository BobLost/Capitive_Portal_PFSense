# Capitive_Portal_PFSense

Documentação Passo a Passo para Implementação do Captive Portal no pfSense
1. Preparação do Ambiente
Certifique-se de ter acesso administrativo ao pfSense via interface web.
2. Configuração do Firewall e Domínio
Acesse System -> General Setup e configure o hostname e domínio conforme seu ambiente.
3. Instalação dos Pacotes Squid e SquidGuard
Vá para System -> Package Manager -> Available Packages.
Instale Squid e SquidGuard.
4. Atualização da Integração de Autenticação do Captive Portal
Substitua o conteúdo do arquivo /usr/local/bin/check_ip.php pelo script de verificação de IP.

#!/usr/local/bin/php-cgi -q
<?php
/*
 * check_ip.php
 *
 * part of pfSense (https://www.pfsense.org)
 * Copyright (c) 2016-2022 Rubicon Communications, LLC (Netgate)
 * Copyright (c) 2013-2016 Marcello Coutinho
 * All rights reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

require_once("config.inc");
require_once("globals.inc");
error_reporting(0);
global $config, $g;

// stdin loop
define("STDIN", fopen("php://stdin", "r"));
define("STDOUT", fopen('php://stdout', 'w'));
while (!feof(STDIN)) {
	$check_ip = preg_replace('/[^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}]/', '', fgets(STDIN));
	$status = '';

       $db = "/usr/local/etc/squid/users.db";
       $status = squid_check_ip($db, $check_ip);

       if ($status) {
               fwrite(STDOUT, "OK user={$status}\n");
       } else {
               fwrite(STDOUT, "ERR\n");
	}

}

function squid_check_ip($db, $check_ip) {
       exec("/usr/local/bin/sqlite3 {$db} \"SELECT ip FROM users WHERE ip='{$check_ip}'\"", $ip);
	if ($check_ip == $ip[0]) {
               exec("/usr/local/bin/sqlite3 {$db} \"SELECT username FROM users WHERE ip='{$check_ip}'\"", $user);
		return $user[0];
	}
}

?>


   
