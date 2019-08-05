# AVCP-Xml - Repository di sviluppo
Generatore di dataset XML per l'Autorità per la Vigilanza sui Contratti Pubblici - art.32 L. 190/2012

## Upgrade dalla versione 0.7.1
Per aggiornare il programma, [scaricare la release più recente](https://github.com/provinciadicremona/AVCP-Xml/releases/latest) e seguire le istruzioni che trovate in quella pagina.

## API per l'ultima versione disponibile
`https://api.github.com/repos/provinciadicremona/AVCP-Xml/releases/latest`

### Aggiunta campo a tabella lotti

```sql
ALTER TABLE `avcp_lotto` ADD `chiuso` BOOLEAN NOT NULL DEFAULT FALSE AFTER `flag`;
```

Aggiungo il campo `chiuso` per permettere all'utente di segnalare come conclusi i lotti per cui non viene pagata l'intera somma aggiudicata.

### Aggiunta vista per esportazione ods dei lotti utente

```sql
CREATE VIEW `avcp_export_ods` AS select 
    `l`.`id` AS `id`,
    `l`.`anno` AS `anno`,
    `l`.`numAtto` AS `numAtto`,
    `l`.`cig` AS `cig`,
    `l`.`oggetto` AS `oggetto`,
    `l`.`sceltaContraente` AS `sceltaContraente`,
    `l`.`dataInizio` AS `dataInizio`,
    `l`.`dataUltimazione` AS `dataUltimazione`,
    `l`.`importoAggiudicazione` AS `importoAggiudicazione`,
    `l`.`importoSommeLiquidate` AS `importoSommeLiquidate`,
    (select count(0) 
        from `avcp_ld` `ldl` 
        where ((`l`.`id` = `ldl`.`id`) 
            and (`ldl`.`funzione` = '01-PARTECIPANTE'))) AS `partecipanti`,
    (select count(0) from `avcp_ld` `ldl` 
        where ((`l`.`id` = `ldl`.`id`) 
            and (`ldl`.`funzione` = '02-AGGIUDICATARIO'))) AS `aggiudicatari`,
    `l`.`userins` AS `userins`,
    group_concat(`ditta`.`ragioneSociale` separator 'xxxxx') AS `nome_aggiudicatari` 
    from ((`avcp_lotto` `l` 
            left join `avcp_ld` `ld` 
            on(((`l`.`id` = `ld`.`id`) 
                    and (`ld`.`funzione` = '02-AGGIUDICATARIO')))) 
        left join `avcp_ditta` `ditta` 
        on((`ld`.`codiceFiscale` = `ditta`.`codiceFiscale`))) 
    group by `l`.`id` 
    order by `l`.`anno`,
    `l`.`id`;

```
