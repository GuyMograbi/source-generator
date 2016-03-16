# source-generator

A project that aims to make source generation for multiple languages from configuration easy.

For example - lets say you have a REST API. 
Now you probably want to have clients for this API in multiple languages - python, ruby, java, javascript and more. 

You could simply maintain multiple different projects, but the logic will suddenly seem repeated. 
open an HTTP connection, send parameters, propegate response. 
and what about testing each client? 


An alternative to this would be to automatically generate the sources. 

# Example

```

nodes:
  list: 
    params:
      - deployment_id
      - node_id
      - _include
  get:
    params:
      - deployment_id
      - node_id
      - _include

```

will generate the following in python (for example. the template is customized)

```
class NodesClient(Client):
  def list(self, deployment_id=None, _include=None, **kwargs):
    params = {}
    if deployment_id:
      params['deployment_id'] = deployment_id
    params.update(kwargs)
    if not params:
      params = None
    response = self.api.get('/nodes', params=params, _include=_include)
    return ListResponse([Node(item) for item in response ['items']], response['metadata'])
  
  def get(self, deployment_id, node_id, _include=None):
    assert deployment_id
    assert node_id
    result = self.list(deployment_id=deployment_id,
                        id = node_id, 
                        _include=_include)
    if not result:
      return None
    else:
      return result[0]

```

and the following in javascript (again, the output is customizable)

```

function NodesClient( config ){
    this.config = config;
}

NodesClient.prototype.list = function( deployment_id, node_id, _include , callback ){
    logger.trace('listing nodes');
    var qs = {};

    if ( deployment_id ){
        qs.deployment_id = deployment_id;
    }

    if ( node_id ){
        qs.id = node_id;
    }

    return this.config.request(
        {
            'method' : 'GET',
            'json': true,
            'url' : this.config.endpoint + '/nodes',
            qs: qs
        },
        callback
    );
};


NodesClient.prototype.get = function( deployment_id, node_id, _include, callback ){
    logger.trace('getting nodes');
    if ( !deployment_id ){
        callback( new Error('deployment_id is missing'));
        return;
    }

    if ( !node_id ){
        callback( new Error('node_id is missing'));
        return;
    }

    this.list( deployment_id, node_id, _include, function( err, response, body ){
            if ( !!body && body.length > 0 ){
                body = body[0];
            }
            callback(err, response, body);
        }
    );
};

module.exports = NodesClient;
```

# Customizing

This project will come with some very basic implementation for a function. 

It is aimed to get you going but not intend to deliver a final output. 

The project assumes you will write your own template based on your own standards. 

