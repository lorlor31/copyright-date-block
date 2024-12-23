# CREER UN BLOC GUTENBERG

Basé sur https://developer.wordpress.org/block-editor/getting-started/tutorial/

## Quelques concepts sur le fonctionnement des blocs
L'enregistrement = registration se fait côté client et côté serveur (cf . https://developer.wordpress.org/block-editor/getting-started/fundamentals/registration-of-a-block/)

## Installer create-block
Create-block va installer un package node pour avoir la structure du plugin qi contiendra le bloc.
Créer un répertoire qui contiendra le plugin et dedans, lancer la commande : 
` npx @wordpress/create-block@latest copyright-date-block --variant=dynamic `
Le plugin sera disponible au nom **Copyright Date Block** ds le tableau de bord admin et ds Gutenberg.
Un dossier Copyright Date Block a été créé dans /plugins.
```
Commandes utiles disponibles pour Create-block

  $ npm start
    Starts the build for development.

  $ npm run build
    Builds the code for production.

  $ npm run format
    Formats files.

  $ npm run lint:css
    Lints CSS files.

  $ npm run lint:js
    Lints JavaScript files.

  $ npm run plugin-zip
    Creates a zip file for a WordPress plugin.

```

Aller dans le dossier du bloc Copyright Date Block  et lancer `npm run start` pour voir les changements en temps réél.

## Structure du bloc

https://developer.wordpress.org/block-editor/getting-started/fundamentals/file-structure-of-a-block/

### block.json ds /src 

```
{
    "$schema": "https://schemas.wp.org/trunk/block.json",
    "apiVersion": 3,
    "name": "create-block/copyright-date-block",
    "version": "0.1.0",
    "title": "Copyright Date Block",
    "category": "widgets",
    "icon": "smiley", // Icons de Dashicons
    "description": "La description de mon bloc",
    "example": {},
    "supports": { // Options dispos pour le style du bloc 
        "html": false
    },
    "textdomain": "copyright-date-block",
    "editorScript": "file:./index.js",
    "editorStyle": "file:./index.css", //style côté frontend
    "style": "file:./style-index.css", //style côté frontend
    "render": "file:./render.php",
    "viewScript": "file:./view.js" /:style côté frontend
}
```
- **icon** : On peut utiliser les icônes de Dashicons https://developer.wordpress.org/resource/dashicons/#pdf. 
On peut enlever la ligne "icon" si on veut par exemple mettre un svg à la place.
Il faudra le faire dans le fichier index.js (cf ci-dessous)
- **supports** : 
On peut rajouter des options pour que l'utilisateur customise le style du bloc. Des sortes de propriétés CSS.
```
"supports": {
    "color": {
        "background": false,
        "text": true
    },
    "html": false,
    "typography": {
        "fontSize": true
    }
},

```
Doc détaillée : https://developer.wordpress.org/block-editor/reference-guides/block-api/block-supports/
Si le style est juste géré par les supports, on peut enlever les lignes editorStyle, style et viewScript du block.json.
Et de même, on pourra enlever les lignes
```
In the edit.js file, remove the lines that import editor.scss
In the index.js file, remove the lines that import style.scss
Delete the editor.scss, style.scss, and view.js files
```

### index.js ds /src 
Fichier js principal, sert notamment à l'enregistrement côté client avec la fonction `registerBlockType`.
Là on a personnalisé l'icône avec un svg.
```
const calendarIcon = (
    <svg
        viewBox="0 0 24 24"
        xmlns="http://www.w3.org/2000/svg"
        aria-hidden="true"
        focusable="false"
    >
        <path d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm.5 16c0 .3-.2.5-.5.5H5c-.3 0-.5-.2-.5-.5V7h15v12zM9 10H7v2h2v-2zm0 4H7v2h2v-2zm4-4h-2v2h2v-2zm4 0h-2v2h2v-2zm-4 4h-2v2h2v-2zm4 0h-2v2h2v-2z"></path>
    </svg>
);

registerBlockType( metadata.name, {
    icon: calendarIcon,
    edit: Edit
} );

```

### edit.js ds /src 
Contrôle l'apparence dans l'éditeur Gutenberg.
- useBlockProps() correspond aux classes et styles requises par l'éditeur
```
export default function Edit() {
    const currentYear = new Date().getFullYear().toString();

    return (
        <p { ...useBlockProps() }>© { currentYear }</p>
    );
}
```
Là on a déclaré une constante qu'on va afficher 

### render.php ds /src 
En modifiant le fichier edit.js, le bloc est modifié dans l'interface de l'éditeur mais par contre dans le rendu de la page, on n'aura pas encore le contenu modifié.

```
<?php
...
?>
<p <?php echo get_block_wrapper_attributes(); ?>>© <?php echo date( "Y" ); ?></p>
```

## Rajouter des attributs de bloc

Les attributs de blocs vont permettre de stocker de la donnée pour le bloc
qui pourra ensuite être utilisée pour modifier le balisage du bloc.
L'exemple se base sur l'ajout d'une date de début au bloc Copyright.

- Dans block.json, on rajoute les attributs :
```
"attributes": {
    "showStartingYear": { // savoir si on affiche cette date de début
        "type": "boolean"
    },
    "startingYear": { //attribut de la date de début
        "type": "string"
    }
},
```

- Dans edit.js, on va rajouter des contrôles de réglages dans l'UI grâce au composant **InspectorControls**.
1. On va l'importer : 
` import { InspectorControls, useBlockProps } from '@wordpress/block-editor'; ` 
2. On va aussi importer des Core Components pour customiser notre UI :
` import { PanelBody, TextControl, ToggleControl } from '@wordpress/components'; `
3. On va créer un fragment `<></>`qui contiendra le JSX 

```
export default function Edit() {
    const currentYear = new Date().getFullYear().toString();

    return (
        <>
            <InspectorControls>
                <PanelBody title={ __( 'Settings', 'copyright-date-block' ) }>
                    Testing
                </PanelBody>
            </InspectorControls>
            <p { ...useBlockProps() }>© { currentYear }</p>
        </>
    );
}
```

4. Pour customiser le contrôle du texte, on va utiliser le composant **TextControl**.Il faut utiliser les **attributs** et la fonction **setAttributes()** en paramètre de edit() .Et on va déclarer les attributs avec `const { showStartingYear, startingYear } = attributes;` .
5. On utilisera ensuite notre composant <TextControl/> qui aura : 
- un label :  “Starting year”
- une valeur : qui prendra la valeur de l'attribut `startingYear`
- onChange avec une fonction qui mettra à jour `startingYear` quand la valeur du texte changera 

```
export default function Edit( { attributes, setAttributes } ) {
    const { showStartingYear, startingYear } = attributes;
    const currentYear = new Date().getFullYear().toString();

    return (
        <>
            <InspectorControls>
                <PanelBody title={ __( 'Settings', 'copyright-date-block' ) }>
                    <TextControl
                        __nextHasNoMarginBottom
                        __next40pxDefaultSize
                        label={ __(
                            'Starting year',
                            'copyright-date-block'
                        ) }
                        value={ startingYear || '' }
                        onChange={ ( value ) =>
                            setAttributes( { startingYear: value } )
                        }
                    />
                </PanelBody>
            </InspectorControls>
            <p { ...useBlockProps() }>© { currentYear }</p>
        </>
    );
}
```
Cela a permis d'enregistrer la valeur tapée dans le champ Text dans les données du bloc. 

6. On va passer au **Toggle**.
On va utiliser le composant <ToggleControl/>. De la même manière, on configure : 
- un label :  "Show starting year”
- un checked : qui prendra la valeur de l'attribut `showStartingYear`
- onChange avec une fonction qui mettra à jour `showStartingYear` quand on cliquera sur le toggle
- on pourra n'afficher l'input text que si le checked est à true avec l'opérateur logique &&. 

```

export default function Edit( { attributes, setAttributes } ) {
    const { showStartingYear, startingYear } = attributes;
    const currentYear = new Date().getFullYear().toString();

    return (
        <>
            <InspectorControls>
                <PanelBody title={ __( 'Settings', 'copyright-date-block' ) }>
                    <ToggleControl
                        checked={ !! showStartingYear }
                        label={ __(
                            'Show starting year',
                            'copyright-date-block'
                        ) }
                        onChange={ () =>
                            setAttributes( {
                                showStartingYear: ! showStartingYear,
                            } )
                        }
                    />
                    { showStartingYear && (
                        <TextControl
                            __nextHasNoMarginBottom
                            __next40pxDefaultSize
                            label={ __(
                                'Starting year',
                                'copyright-date-block'
                            ) }
                            value={ startingYear || '' }
                            onChange={ ( value ) =>
                                setAttributes( { startingYear: value } )
                            }
                        />
                    ) }
                </PanelBody>
            </InspectorControls>
            <p { ...useBlockProps() }>© { currentYear }</p>
        </>
    );
}
```
Le toggle se met bien à jour.

7. Mettre à jour le contenu du bloc
- On crée la logique pour afficher la date de début si elle existe avecla variable displayDate .
```
   let displayDate;

    if ( showStartingYear && startingYear ) {
            displayDate = startingYear + '–' + currentYear;
    } else {
        displayDate = currentYear;
    }
```
- On met à jour la fonction edit.js pour utiliser la nouvelle variable displayDate.

8. Mettre à jour le rendu de la page dans render.php
On va faire ce qu'on a fait dans edit.js, déclarer la variable et l'utiliser
```
if ( ! empty( $attributes['startingYear'] ) && ! empty( $attributes['showStartingYear'] ) ) {
    $display_date = $attributes['startingYear'] . '–' . $current_year;
} else {
    $display_date = $current_year;
}
```
donc render.php sera finalement : 
```
<?php
$current_year = date( "Y" );

if ( ! empty( $attributes['startingYear'] ) && ! empty( $attributes['showStartingYear'] ) ) {
    $display_date = $attributes['startingYear'] . '–' . $current_year;
} else {
    $display_date = $current_year;
}
?>
<p <?php echo get_block_wrapper_attributes(); ?>>
    © <?php echo esc_html( $display_date ); ?>
</p>
```

## Rajout d'un rendu statique 
https://developer.wordpress.org/block-editor/getting-started/tutorial/#adding-static-rendering
Un bloc peut utiliser le rendu statique, dynamique ou les deux. Là le bloc est rendu dynamiquement : les données ( attributs des balises etc) sont enregistrées ds la BDD mais pas le rendu final en HTML.
Le rendu statique va permettre de sauvegarde rle bloc en BDD et si on désinstalle le plugin du bloc, on ne perdra pas les données. => c'est plus prudent et pérenne

1/ Utilisation de la fonction save()
- On va créer un fichier save.js avec 
```
import { useBlockProps } from '@wordpress/block-editor';

export default function save() {
    return (
        <p { ...useBlockProps.save() }>
            { 'Copyright Date Block – hello from the saved content!' }
        </p>
    );
} 
```
- Dans index.js, on importe cette fonction  : `import save from './save';
` et on rajoute save dans registerBlockType.
- On aura des erreurs en front comme le HTML du bloc n'est pas enregistré en BDD.
- On va créer la fonction save en s'inspirant de edit()
```
export default function save( { attributes } ) {
    const { showStartingYear, startingYear } = attributes;
    const currentYear = new Date().getFullYear().toString();

    let displayDate;

    if ( showStartingYear && startingYear ) {
        displayDate = startingYear + '–' + currentYear;
    } else {
        displayDate = currentYear;
    }

    return (
        <p { ...useBlockProps.save() }>© { displayDate }</p>
    );
}
```
- Mais un problème se posera si on essaie de mettre à jour ultérieurement, le HTML en BDD sera contradictoire avec les données générées dynamiquement donc il faut rajouter un attribut fallbackCurrentYear qui va nous permettre de gérer les contradictions. On remplacera currentYear par cet attribut dans save (cf ficher save.js final). 
- Il faut veiller aussi à ne pas rendre de HTML si on n'a pas de fallbackCurrentYear, revoir toute la partie 
https://developer.wordpress.org/block-editor/getting-started/tutorial/#handling-dynamic-content-in-statically-rendered-blocks
