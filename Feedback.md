# React style guide

This is an opinionated style guide for medium-size projects powered by [create-react-app](https://github.com/facebook/create-react-app) and [react-apollo](https://github.com/apollographql/react-apollo).

Check this project for an example: [paulnta/github-issues-viewer](https://github.com/paulnta/github-issues-viewer)

## Folder Structure 

```
/src
  /components
    InputSearch.js
    IssueList.js
    IssueListItem.js
    IssueComment.js
    RepoListItem.js
    ...
  /routes
    IssuePage.js
    IssuesPage.js
    SearchPage.js
    index.js
    ...
  /stories
  /svgIcons
  /lib
    /utils
      time-helpers.js
      ...
  App.js
  index.js
```

`/components` 

This directory contains all React components that might be used in two or more views. Child components that are tightly coupled with their parent include the parent component name as a prefix.

`/routes`

This directory, often called views or screens, contains Layout components for our pages. They are in general single-instance components. They often rely on url parameters to fetch data.

`/svgIcons` 

Contains custom SVG icons.

`/lib`

It is common to have a folder that contains code that is note entirely related to React. It allows you to put some helpers, validations etc..

## Component

Components are divided in two categories — Containers and Presentational components. However, they are kept in the same `components` folder.



#### Presentational components – *How things look*

- Use their own styling
- Avoid when possible any dependencies on the rest of the app, such as react-router or react-apollo.
- Rarely have their own state
- Are fully reusable and can be easily integrated in your storybook.
- Are written as functional components



For example the following component `IssueListItem` looks like a traditional template. It is concerned only about presentation. 

```jsx
import React from "react";
import PropTypes from "prop-types";
import cx from 'classnames';
import { withStyles } from '@material-ui/core/styles';
import { timeAgo } from '../utils';
import { IssueState } from "../constants";

const styles = theme => ({
  title: {
    fontWeight: 500,
    color: theme.palette.text.primary,
  },
});

const IssueListItem = ({ classes, className, title, number, createdAt, state, author, commentCount, loading }) => (
  <div className={cx(classes.root, className)}>
    <div className={classes.title}>{state} - {title}</div>
    <div>#{number} opened {timeAgo(createdAt)} by {author}</div>
    <div>{commentCount} comments</div>
  </div>
);

IssueListItem.propTypes = {
  classes: PropTypes.objectOf(PropTypes.string),
  className: PropTypes.string,
  title: PropTypes.string,
  number: PropTypes.number,
  state: PropTypes.oneOf([IssueState.OPEN, IssueState.CLOSED]),
  author: PropTypes.string,
  createdAt: PropTypes.string,
  commentCount: PropTypes.number,
};

export default withStyles(styles)(IssueListItem);
```



#### Containers – *How things work*

- Use state and lifecycle methods if needed.
- Fetch data using react-apollo
- Have dependencies on libraries such as react-apollo or react-router-dom



The following container component `IssueList` is tasked with fetching data (using react-apollo) and then rendering the related view component. It is also responsible to implement the logic for navigation using `Link` from example react-router-dom. 

```jsx
// IssueList.js
import React from 'react';
import PropTypes from 'prop-types';
import { Query } from 'react-apollo';
import gql from 'graphql-tag';
import { Link } from 'react-router-dom';
import IssueListItem from './IssueListItem';

const IssueList = ({ owner, name, state }) => {
  return (
    <Query
      query={gql`...`}
      variables={{ owner, name, state }}
    >
      {({ loading, data, error }) => {
        if (loading) return 'Loading...';
        if (error) return 'Error...';
        return (
          <div>
            {data.repository.issues.edges.map(({ node: issue }) => (
              <Link key={issue.id} to={`/${owner}/${name}/issues/${issue.number}`}>
                <IssueListItem
                  number={issue.number}
                  title={issue.title}
                  author={issue.author ? issue.author.login : undefined}
                  createdAt={issue.createdAt}
                  commentCount={issue.comments.totalCount}
                  state={issue.state}
                />
              </Link>
            ))}
          </div>
        );
      }}
    </Query>
  );
};

IssueList.propTypes = {
  owner: PropTypes.string.isRequired,
  name: PropTypes.string.isRequired,
  state: PropTypes.string,
};

export default IssueList;
```


## Storybook

> Storybook is a development environment for UI components. It allows you to browse a component library, view the different states of each component, and interactively develop and test components.

Similary to how unit testing works, storybook allows you to see and interact with your components in an isolated way.

- Allows you to test different scenario that can be really difficult to test from your app.
- It helps you implement a uniform styling for all your UI components
- It serves as documentation/library for your team



#### Tips

With storybook it is possible to decorate components with anything we want.

The `addDecorator` method accepts a function that returns a component. This API is super useful when we need some wrapping styles or allow using react-router-dom components.



```jsx
import React from 'react';
import { configure, addDecorator } from '@storybook/react';
import Theme from '../src/components/Theme';
import CssBaseline from '../src/components/CssBaseline';
import ThemeToggle from '../src/components/ThemeToggle';

function loadStories() {
  require('../src/stories');
}

// This component will wrap all my stories
// similary to what we use to wrap our App component.
addDecorator(story => (
  <Theme>
    <CssBaseline>
      <ThemeToggle />
      {story()}
    </CssBaseline>
  </Theme>
))

configure(loadStories, module);
```
